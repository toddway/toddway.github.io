On projects where multiple systems undergo development at the same time, it's crucial to maintain a clear picture of how they should interact.  We commonly have a backend system providing a REST API to multiple frontends (browsers, mobile apps, chatbots, IoT, etc.).  Because it will likely change over time, keeping the API picture clear and up to date can be a significant challenge.  How can we efficiently describe the currently expected behavior and know if it's working as expected so teams don't spin their wheels due to miscommunication?

## Cucumber 
[Cucumber](http://cucumber.io) helps us write readable requirements upfront that can be tied directly to executable tests.  Here's an example for a guestbook REST API:


	Scenario: Read a list of guestbook entries
	
	  Given I'm using the staging API environment
	  And the guestbook has at least "2" entries
	  When I make a GET request to "/guestbook/entries"
	  Then I get a response code of "200"
	  And I get a response with at least "2" entries
	  And each entry has a "name"
	  And each entry has a "date" formatted as a Unix timestamp

Each line in the scenario above represents a discrete Cucumber step.  A developer can now write a short block of code to fulfill each step.  

There are options in various languages for fulfilling Cucumber step definitions (e.g. Ruby, Javascript, Python, .NET, Java).  I chose the Java implementation, [Cucumber-JVM](https://github.com/cucumber/cucumber-jvm), for these reasons:

1. Works with many out-of-the-box **reporting and automation tools** - because it's JUnit-based
2. Intuitive **IDE support** for code assist, breakpoints, debugging, output formatting, etc. (Intellij and Eclipse)
3. Easy-to-build **HTTP request and response assertions** using the [Rest-assured](http://rest-assured.io/) library

## Cucumber on Java
Using Cucumber-JVM and the Intellij IDE, I get automatically generated step definitions like this:
	
	public GuestbookStepDefinitions() {
		Given("^I'm using the staging API environment$", () -> {
			//short block of code goes here
		});
		
		When("^I make a GET request to \"([^\"]*)\"$", (String path) -> {
			//another block here
		});
		
		Then("^I get a response code of \"([^\"]*)\"$", (Integer code) -> {
			//and another
	    });
	}

Next we fill in the implementations using Rest-assured...

## Rest-assured

	public GuestbookStepDefinitions() {
	
	    private RequestSpecification request;
	    private ValidatableResponse response;
	    
	    @Before
	    public void before(Scenario scenario) {
	        request = RestAssured.with();
	    }
	
		Given("^I'm using the staging API environment$", () -> {
			request.given()
		        .contentType(ContentType.JSON)
		        .baseUri("https://staging.mycompany.com");
		});
		
		When("^I make a GET request to \"([^\"]*)\"$", (String path) -> {
			response = request.get(path + ".json").then();
		});
		
		Then("^I get a response code of \"([^\"]*)\"$", (Integer code) -> {
			response.statusCode(code);
	    });
	}


The `Given` and `When` steps are building a request with details for our REST API.  The `Then` step calls `response.statusCode(...)` which is an assertion of the status code returned by the REST API.  If any step fails we get targeted feedback like this:


	java.lang.AssertionError: 1 expectation failed.
	Expected status code  but was .
	
	
	...
	io.restassured.internal.ValidatableResponseOptionsImpl.statusCode(ValidatableResponseOptionsImpl.java:117)
		at GuestbookStepDefinitions.lambda$new$8(GuestbookStepDefinitions.java:66)
		at ✽.Then I get a response code of "100"(guestbook-entries-read.feature:12)
	
	Failed scenarios:
	guestbook-entries-read.feature:9 # Scenario: Read a list of guestbook entries
	
	1 Scenarios (1 failed)
	3 Steps (1 failed, 2 passed)
	0m1.370s


This output is a bit verbose (we'll worry about report formatting later), but contains important information about the failure.  

The step: we see that the "Then I get a response code of 100" step of our "Read a list of guestbook entries" scenario is where we're failing.  That means the previous two steps passed successfully.  

The expectation:  we see that we got a  response code but expected a  response code.  If we change the expected status code back to 200, we should get a passing test:

	Scenario: Read all guestbook entries          # guestbook-entries-read.feature:9
	  Given I'm using the staging API environment # GuestbookStepDefinitions.java:89
	  When I make a GET request to "/guestbook"   # GuestbookStepDefinitions.java:59
	  Then I get a response code of "200"         # GuestbookStepDefinitions.java:65
	
	1 Scenarios (1 passed)
	3 Steps (3 passed)
	0m1.937s


Since we're making HTTP calls it'd be nice to see the request and response details too.  We can tell Rest-assured to print those along with our test results:


	Request method:	GET
	Request URI:	https://staging.mycompany.com/guestbook/entries.json
	Proxy:			<none>
	Request params:	<none>
	Query params:	<none>
	Form params:	<none>
	Path params:	<none>
	Multiparts:		<none>
	Headers:		Accept=*/*
					Content-Type=application/json; charset=UTF-8
	Cookies:		<none>
	Body:			<none>
	HTTP/1.1 200 OK
	Server: nginx
	Date: Mon, 10 Apr 2017 19:09:16 GMT
	Content-Type: application/json; charset=utf-8
	Content-Length: 4830
	Connection: keep-alive
	Access-Control-Allow-Origin: *
	Cache-Control: no-cache
	Strict-Transport-Security: max-age=31556926; includeSubDomains; preload
	{
	    "-KgBbHUcv2NWn2M6tzGp": {
	        "comment": "Hello Guestbook",
	        "name": "Test User",
	        "timestamp": 1490565277672
	    },
	    "-KgBbzZE2WtRD9wz1t-D": {
	        "comment": "Hello Guestbook",
	        "name": "Test User",
	        "timestamp": 1490565462287
	    }
	}


## Reporting
Now when we run this test we immediately know three things:
1. **What we expect to happen** (the Given-When-Then statement)
2. **How to make it happen** (the printed request and response)
3. **Is it currently working as expected** (Pass or Fail)

Having this feedback continuously throughout development mitigates communication issues early before teams waste time heading in different directions.  The easy-to-read Cucumber steps that everyone can read tie directly to the gritty HTTP definitions that developers need and we can drop it on a CI server to generate formatted reports visible to the whole team.

Here's an example of formatted results from the Intellij IDE:
<figure class="tmblr-full" data-orig-height="728" data-orig-width="1188" data-orig-src="https://sandbox-9221c.firebaseapp.com/blog/ide-cucumber-output.png"><img src="https://64.media.tumblr.com/2b9505cb00c137a2246a8f353a981c3d/tumblr_inline_pjzq05TyS81r4ik0y_540.png" width="600px" data-orig-height="728" data-orig-width="1188" data-orig-src="https://sandbox-9221c.firebaseapp.com/blog/ide-cucumber-output.png"></figure>
On the left we have a collapsible, colored outline of our features, scenarios, and steps.  We can select anything in the tree and see corresponding details on the right.

And here's a standalone HTML report:
<figure class="tmblr-full" data-orig-height="1410" data-orig-width="1896" data-orig-src="https://sandbox-9221c.firebaseapp.com/blog/web-cucumber-output.png"><img src="https://64.media.tumblr.com/116c8b133d374021376065b667d1341d/tumblr_inline_pjzq06uSaT1r4ik0y_540.png" width="600px" data-orig-height="1410" data-orig-width="1896" data-orig-src="https://sandbox-9221c.firebaseapp.com/blog/web-cucumber-output.png"></figure>
Again we have a collapsible, colored outline that documents the expected behavior and HTTP details.


## Process
This approach is designed drive collaboration early in the process so it's a great chance to work in pairs.  Pairing a frontend developer and a backend developer can help start the conversation about how systems should interact.  Getting other roles like analysts, designers, and testers involved can level-set everyone's understanding of how the product is supposed to work.   As soon as we have requirements for our first feature, we can start writing tests.  The code required to fulfill step definitions should be easy enough for any developer to pick up quickly regardless of language choice.  I prefer to put API acceptance tests in a separate repository apart from any other production code.   This limits external dependencies from affecting our ability to write and run the tests.
