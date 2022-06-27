In a [previous post](http://toddway.com/post/165619029205/types-and-tests-for-firebase-cloud-code) I showed how to add type-checking and unit tests to Firebase cloud code (cloud functions *and* database rules).  Those tests are independent from any Firebase environment and independent from each other.   We should be able to run them all quickly and consistently in a clean Node.js environment with a single command.  We should also be able to chain that command with others so that we can build, test, and deploy each code commit directly into a live Firebase environment.


#### Here are the steps we want to automate:

1. Download project source
2. Download project dependencies
3. Compile project
4. Run all tests
5. Stop if any test fails, otherwise continue
6. Deploy (cloud functions and database rules) to a Firebase environment
7. Write a deployment summary to our Firebase environment (Git info, date, test results)


We'll assume we have a machine with Node.js and Git installed.  This could be a developer machine or a dedicated continuous integration server (I try to make the execution identical for either if I can).  The first three steps are pretty straightforward from the command line:

	git clone git@github.com:whatever folder-name
	npm install
	tsc
	
Now that we have a compiled project environment, we can use Typescript/Javascript to handle the rest of our steps.  From the command line, node can execute a function from a local file like this:

	node -e 'require("./build.js").runAllTests()'

Now let's implement a function to run all our tests:

	export async function runAllTests() : Promise<testresultsentity> {
	    const Mocha = require('mocha');
	    const mocha = new Mocha();
	    mocha.addFile('./test/tests.functions.js');
	    mocha.addFile('./test/tests.database.js');
	    const results = new TestResultsEntity();
	    return await new Promise<testresultsentity>((resolve, reject) => {
	        mocha.run()
	            .on('pass', (test) => { results.passed++; })
	            .on('fail', (test, err) => { results.failed++; })
	            .on('end',  () => { resolve(results) });
	    });
	}

Here I'm use the Mocha API to point to our test files, run them, and keep track of how many pass and fail.  Let's write a deploy function that grabs the test results and handles our last three steps:

	export async function deploy()  {
	    const testResults = await runAllTests();
	    const gitSummary = await getGitSummary();
	    const summary = `${testResults.getSummary()} on ${getDateSummary()} from ${gitSummary}`;
	
	    if (testResults.hasFailures()) {
	        console.log('Deploy failed');
	    } else {
	        await asyncCommand(`firebase deploy --only functions,database`);
	        await asyncCommand(`firebase database:set /lastDeploy -d '"${summary}"' -y`);
	        console.log('Deploy succeeded');
	    }
	
	    console.log(summary);
	}

In the else clause above we run two [Firebase CLI](https://firebase.google.com/docs/cli/) commands.  The first command deploys our code to the currently configured Firebase environment.  The second command writes a record to the database of that environment with a summary of our deployment.  This makes it easy for anyone on the team to see:

1. what code is deployed, 
2. when it happened, 
3. and the results of the tests.

To pull all of this together into a single command, we'll use the package.json file to set up an npm script:

	"scripts": {
	   "deploy": "npm install && tsc && node -e 'require(\"./build.js\").deploy()'"
	 }

Now we can run all steps with these two commands:

	git clone git@github.com:whatever folder-name
	npm run deploy


Finally, here's the ancillary code referenced by the deploy() and runAllTests() functions above:

	export async function getGitSummary() : Promise<string> {
	    const gitSha = await asyncCommand("git rev-parse --short HEAD");
	    const gitBranch = await asyncCommand("git rev-parse --abbrev-ref HEAD");
	    return Promise.resolve(`${gitSha.trim()}-${gitBranch.trim()}`);
	}
	
	function getDateSummary() : string {
	    return new Date().toLocaleString("en-US", { timeZone: 'America/Chicago' }).trim();
	}
	
	
	const exec = require('child_process').exec;
	function asyncCommand(command : string) : Promise<string> {
	    return new Promise<string>((resolve, reject) => {
	        exec(command, function(error, stdout, stderr){ resolve(stdout); });
	    })
	}
	
	export class TestResultsEntity {
	    passed : number = 0;
	    failed : number = 0;
	
	    getSummary() : string {
	        return `${this.passed}/${this.failed+this.passed} tests passed`
	    }
	
	    hasFailures() : boolean { return this.failed != 0 }
	}


 </string></string></string></testresultsentity></testresultsentity>
