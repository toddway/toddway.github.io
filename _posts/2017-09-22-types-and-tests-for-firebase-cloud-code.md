Firebase makes it cheap and easy to code custom functions and database access rules that run in the cloud along with their standard services.  This is great because you can try things out very quickly in a real, shared environment that performs at scale.  This simplicity make it tempting just to verify each change manually in the cloud environment.  I think static type-checking and independent unit tests are an even more appealing way to verify our code with confidence.  Here's a recipe for eliminating dependencies on Firebase so we can run fast, automated tests before deploying code to the cloud.

## Typescript + IDE
[Typescript](https://www.typescriptlang.org/) is a superset of Javascript that compiles to plain Javascript.  It adds many useful language features like type annotations, interfaces, classes, and generics to Javascript.  All features are optional, so you can always ignore types and interop with plain Javascript when you want.   

Image we want have an app where we want to archive posts if they've been flagged too many times.  Let's use a Typescript interface to decouple the Firebase dependency from as much of our code as we can.

Say we have a PostEntity class like this:


	class PostEntity {
	    id : string;
	    flags : number;
	    maxFlags = 5;
	
	    hasTooManyFlags() : boolean {
	        return this.flags >= this.maxFlags
	    }
	}


...and a function that fetches a post by id, determines if it has too many flags, and archives it if so:

	function archiveIfTooManyFlags(postId : string, datasource : PostDatasource) : Promise<void> {
	    return datasource.getPost(postId)
	        .then(post => {
	            if (post.hasTooManyFlags())
	                return datasource.archivePost(post.id);
	            else
	                return Promise.resolve();
	        })
	}
	
	interface PostDatasource {
	    getPost(postId : string) : Promise<postentity>
	    archivePost(postId : string) : Promise<void>
	}


Notice that we defined a PostDatasource *interface* with methods getPost and archivePost.  We can write an implementation of that interface that uses Firebase as our datasource or we could write an implementation that uses something completely different.  All of the code we've written so far is independent of that implementation.

Here's what that implementation might look like using Firebase:

	class PostDatasourceFb implements PostDatasource {
		db = require('firebase-admin').database();
	
	    getPost(postId: string) : Promise<postentity> {
	        return this.db.ref(`/posts/${postId}`).once('value')
		        .then(snap => snap.val());
	    }
	
	    archivePost(postId: string): Promise<void> {
	        return this.db.ref(`/posts/${postId}/isArchived`).set(true);
	    }
	}


To execute this as a cloud function when a post is flagged in our Firebase Database we do this :

	exports.onFlaged = functions.database.ref('/posts/{postId}/flags')
		.onWrite(event => {
		    return archiveIfTooManyFlags(
			    event.params.postId, 
			    new PostDatasourceFb()
			);
		});



In addition you can use a Typescript-aware IDE to make discovering the properties of your objects, language features and other javascript dependencies automatic as you write code.  If you've used any other OO language with a good IDE you'll know what this means.  [WebStorm](https://www.jetbrains.com/webstorm/) is my choice because I'm already familiar with the shortcuts and organization of JetBrains tools, but there are [many others](https://github.com/Microsoft/TypeScript/wiki/TypeScript-Editor-Support).   They all support features like: instant type checking, code assist, inline refactoring, breakpoint debugging, object navigation, source control tracking, etc.


## Mocha + Chai + Sinon
Now that we have the language tools to decouple our application code from dependencies, we can set up unit tests that run locally outside of the Firebase cloud environment.  Our test libraries are: 

1. [Mocha](https://mochajs.org/) -  lets you describe and execute a set of unit tests.  
2. [Chai](http://chaijs.com/) - lets you make assertions within each of those tests.  
3. [Sinon](http://sinonjs.org/) - lets you create spies, stubs, and mocks for your test dependencies.

Here's what a test for our archiveIfTooManyFlags function might look like:

	describe('archiveIfTooManyFlags', () => {
	    it('should call archivePost when there are too many flags', async () => {
	        const sinon = require('sinon');
	        const post = new PostEntity();
	        post.flags = 7;
	        const postDatasource = <postdatasource>{};
	        postDatasource.getPost = sinon.stub().resolves(post);
	        postDatasource.archivePost = sinon.stub().resolves(null);
	
	        await archiveIfTooManyFlags("123", postDatasource);
	
	        sinon.assert.calledWith(<sinonstub>postDatasource.archivePost, post.id);
	        sinon.assert.calledWith(<sinonstub>postDatasource.getPost, "123");
	    });
	});


First we set up Sinon, and declare a PostDatasource and a PostEntity.  Since PostDatasource is an interface, we have to implement at least the methods we plan to use in our test.  We don't want to use the PostDatasourceFb class from earlier, because it requires a Firebase environment.  We could write a second implementation from scratch that returns some mock values and keeps track of method calls, or we could let Sinon do most of that work for us.  Creating our PostDatasource as {} means it exists, but none of the methods have been implemented yet.  We use sinon.stub() to stub the behavior of the methods we plan to use: getPost and archivePost.  Finally we call archiveIfTooManyFlags and verify that getPost and archivePost were called with the expected arguments.

Now we can run this test and others like it locally without connecting to Firebase.  This covers our cloud functions, but what about our database rules?

## Targaryen
[Targaryen](https://github.com/goldibex/targaryen) lets us write the same kind of Mocha-based unit tests for our Firebase database rules.  

Here's a rule for reading & writing to a post:

	{
	  "rules": {
	    ".read": "false",
	    ".write": "false",
	    "posts" :  {
	      ".read": "true",
	      "$post" : {
	        ".read" : "true",
	        ".write": "auth != null && (data.child('userID').val() == auth.uid || root.child('users/' + auth.uid + '/isAdmin').val() == true)",
	        "views":{
	          ".write":"true"
	        }
	      }
	    }
	}

Only admins and authors should be able to write to (edit) a post.  Targaryen lets us test this rule locally like this:

	describe('posts/aPostId', () => {
	    it(`can write if admin or author`, () => {
	        targaryen.setFirebaseData({
	            users: {
	                adminUser: {
	                    isAdmin:true
	                }
	            },
	            posts: {
	                aPostId: {
	                    userID:"authorUser"
	                }
	            }
	        });
	
	        expect({uid: 'adminUser'}).can.write.path('posts/aPostId');
	        expect({uid: 'authorUser'}).can.write.path('posts/aPostId');
	        expect({uid: 'randomUser'}).cannot.write.path('posts/aPostId');
	        expect(null).cannot.write.path('posts/aPostId');
	    });
	});

Now we can cover all our cloud functions and all our database rules with fast, automated tests! Wouldn't it be great if we had a single command that could confirm all our tests are passing, then deploy the code and results to our Firebase environment? I'll write up my solution to this in a future post.
