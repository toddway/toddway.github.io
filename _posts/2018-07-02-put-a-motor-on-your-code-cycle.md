There's an old programmer joke:

> Show me a line of code and I'll tell you what's wrong with it,
> Show me five hundred lines of code and I'll say "looks ok to me"

Where I work, we use pair programming and pull requests in our development process.  No code is committed to the main branch without peer review.

I'm convinced the investment in extra eyes can pay off for a project of any size.  Human brains are uniquely capable of solving hard problems even when objectives are vaguely defined.  When we do this together it increases ownership, cohesion, velocity and resilience across the team.   This is magical.

But even the collective brainpower of a long-lived, high-functioning team isn't always reliable.  The complexities, pressures, and context-switching of a typical day can wear us down.  When we add the eye-glazing drudgery of a long pull request at 4pm in the afternoon, we're in trouble.

Here's how we're using automation to optimize our review process:

## Build checks
Before anyone reviews source code, a machine should do it first.  It won't catch everything a human could see, but it's more consistent and much faster.  Source control systems (e.g. Github, Bitbucket) usually have an API so build results can be tracked for each code commit.  These are commonly called "status checks" or "build checks".  As shown below, a green icon indicates a check posted from a build server was successful.

<figure class="tmblr-full" data-orig-height="137" data-orig-width="856" data-orig-src="https://sandbox-9221c.firebaseapp.com/blog/passing-check.png"><img src="https://64.media.tumblr.com/3af19c4b589c74f4825c2f263c8bd66e/tumblr_inline_pk09lmqzym1r4ik0y_540.png" data-orig-height="137" data-orig-width="856" data-orig-src="https://sandbox-9221c.firebaseapp.com/blog/passing-check.png"></figure>

Reviewers see a green or red icon immediately and can avoid wasting time reading code with known problems.  Even better, we can make these checks required for merging any code into our main code branch. Now the entire team, whether they review code or not, has confidence that the main branch is *always* protected.

<figure class="tmblr-full" data-orig-height="702" data-orig-width="894" data-orig-src="https://sandbox-9221c.firebaseapp.com/blog/protect-branch.png"><img src="https://64.media.tumblr.com/743da068a6066528fc1f0fa7a7925fc6/tumblr_inline_pk09lnuJZB1r4ik0y_540.png" data-orig-height="702" data-orig-width="894" data-orig-src="https://sandbox-9221c.firebaseapp.com/blog/protect-branch.png"></figure>

We currently use 3 build checks: build, test, and lint. These can work for almost any project.

<figure class="tmblr-full" data-orig-height="235" data-orig-width="472" data-orig-src="https://sandbox-9221c.firebaseapp.com/blog/build-checks.png"><img src="https://64.media.tumblr.com/9e92e39acb0f9bf617fb103499168990/tumblr_inline_pk09loz9uw1r4ik0y_540.png" data-orig-height="235" data-orig-width="472" data-orig-src="https://sandbox-9221c.firebaseapp.com/blog/build-checks.png"></figure>

To process and post the checks, we use a Gradle plugin called [BuildChecks](https://github.com/toddway/BuildChecks).  We use Gradle because it's free, fast, configurable, testable, well-supported, and portable.  From one project to the next, we don't aways get to use the same development languages, source control system, or build servers, but we want to preserve key processes.  BuildChecks can work across multiple languages and source control systems.  It can run anywhere Java 7+ is installed.  In situations where we aren't able to use a dedicated build server, it can be run from a developer's workstation.  We can have the same automated integration protection even on shoestring budgets and timelines.


## Build
The "build" check tells us if a build finished successfully and how long it took.  The process may be different for each project but is typically some variation of: compile source code, assemble artifacts, run tests, run lint, and deploy artifacts.  Having this feedback alone for a pull request review will save considerable time and effort.

## Test
The "test" check show us the percentage of code that is covered by tests.  BuildChecks parses output from coverage tools like JaCoCo, Cobertura, Istanbul, Slather, and OpenCover. A minimum threshold for coverage can be set that will cause the check to fail.  This is optional.  Even without a threshold, the check clarifies that tests are running and if they've changed between commits.

## Lint
The "lint" check tells us if the code violates any predefined standards.  BuildChecks parses output from linters like ESLint, TSLint, Detekt, Checkstyle, PMD, SwiftLint, and Android Lint.  Each linter has different rule sets that span categories like correctness, security, performance, accessibility, formatting style, internationalization, etc.  The lists can be overwhelming at first.  Many are language or platform-specific, but one category that has some pretty universal rules is maintainability.  If you don't know where to start, this is a good place.

Here's list of maintainability rules from a multi-language code analysis platform called [CodeClimate](https://docs.codeclimate.com/docs/maintainability):

- Argument count - Methods or functions defined with a high number of arguments
- Complex logic - Boolean logic that may be hard to understand
- Method complexity - Functions or methods that may be hard to understand
- File length - Excessive lines of code within a single file
- Method count - Classes defined with a high number of functions or methods
- Method length - Excessive lines of code within a single function or method
- Nested control flow - Deeply nested control structures like if or case
- Return statements - Functions or methods with a high number of return statements
- Similar blocks of code - Duplicate code which is not identical but shares the same structure

Enable any of these that your lint tool supports.  Then use pull request conversations to discuss and identify additional patterns you want to add.  If the pattern isn't already available, you can write your own custom rule.

## Details
<figure class="tmblr-full" data-orig-height="235" data-orig-width="472" data-orig-src="https://sandbox-9221c.firebaseapp.com/blog/build-checks.png"><img src="https://64.media.tumblr.com/9e92e39acb0f9bf617fb103499168990/tumblr_inline_pk09loz9uw1r4ik0y_540.png" data-orig-height="235" data-orig-width="472" data-orig-src="https://sandbox-9221c.firebaseapp.com/blog/build-checks.png"></figure>

In the image above, each check has a hyperlink labelled "Details".  This links back to detail on the build server.  It's a great way to give reviewers access to the full generated reports from lint and coverage tools as well as build logs and other artifacts.  The more context we can provide, the easier it will be for others to give real feedback.

## Final thoughts
Automated checks help code reviews scale with consistency. Getting started is as easy as enabling the requirement in your source control system and using a tool like BuildChecks to report it.  If you're not already, this puts you on a path to a several important development practices: automated builds, unit tests, frequent integration, maintainability standards, and protected branches.  Don't worry if you start with low test coverage and high lint violations.  You will immediately have a better understanding of your current situation and a way track your progress.

What automation tricks have you found for improving code review and integration?