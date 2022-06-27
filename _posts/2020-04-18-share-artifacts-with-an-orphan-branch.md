---
layout: post
---

Over the last several years I've worked with a variety of great hosted services for team code integration:

- For build automation - Jenkins, Bitrise, CircleCi, BuddyBuild, TeamCity.
- For code analysis - SonarQube, Codacy, Code Climate, Veracode.
- For peer review - Github, Bitbucket, GitLab, Gerrit.

When everything works in harmony (checking out files, compiling, running tests, analyzing code structure, assembling artifacts, posting clear results) the whole team sees steady feedback, early in the development cycle, that helps prevent defects, educate new team members, and nudge us all to better design habits.

But for various reasons we can't always set up the perfect recipe of hosted services for every project. If information that we've counted on in the past is missing, unreliable, or hard to access, the impact quickly fades. In the worst cases, the [lack of feedback makes us falsely confident](https://youtu.be/ddPQAJSm2cQ).

There are fancy ways to connect build automation services and code analysis services to deployment services and code review services. If these options are available, if you can quickly troubleshoot interruptions, and if your whole team can access the results, then do this.  If not, here is a relatively independent alternative:

Most software platforms have standalone tools for code analysis (test execution, test coverage, linting, docgen, etc.) that can generate standalone reports (probably HTML). If you get familiar with how to generate these, you can port them to future projects without relying on an external hosted service or special IDE features. Make a one-step script that generates all of these artifacts.

Make another one-step script that does this: check out an [orphan branch](https://git-scm.com/docs/git-checkout#Documentation/git-checkout.txt---orphanltnewbranchgt) from your code repository into a temporary folder. Remove existing files. Copy in the generated files from your previous script. Commit and push. Print a download link for the commit.

Finally copy/paste the download link into your pull request description.  Your team can use this link to easily download and browse results.  One additional file I include is a root index.html with short descriptions and relative links for each report.  This helps everyone know why each report is there and gives a single entry point for everything in the download.

For reference, I've set this up on one of my small, public Github projects.  You can see an example [pull request](https://github.com/toddway/Shelf/pull/6) that includes a direct link to [download reports](https://github.com/toddway/Shelf/archive/07e990cb8840e83782d28f9135a25cf75e040ad3.zip).  You can also see the [shell script for pushing artifacts](https://github.com/toddway/Shelf/blob/multiplatform/shelf/push-artifacts.sh) and the [shell script for initializing the branch with an index.html](https://github.com/toddway/Shelf/blob/multiplatform/shelf/push-artifacts-init.sh).  The goal for these scripts is to be relatively generic and portable so they can quickly be applied to future projects.  The same goal applies to scripts that generate report artifacts, but this is inherently more platform-dependent.  In this example, [the script to generate artifacts is Gradle-based](https://github.com/toddway/Shelf/blob/multiplatform/shelf/checks.gradle) and the code analysis tools include JaCoCo, CPD, Detekt, BuildChecks, and JUnit.