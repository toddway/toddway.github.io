In 1935, Boeing introduced a heavy bomber that outperformed any other aircraft of it's kind. It crashed tragically in exhibition because a routine step was missed by the well-trained and experienced flight crew.  As a result, pilots everywhere began to adopt preflight checklists and failures decreased significantly.

Today, checklists for complex responsibilities are used in many professions. A surgeon named Adul Gawande wrote an entire book called [The Checklist Manifesto](http://atulgawande.com/book/the-checklist-manifesto/).   I think one reason the idea has spread so successfully is because a checklist is an incredibly easy tool to make and to use.
[[MORE]]

I've written previously about how we do [continuous](http://toddway.com/post/175477173505/put-a-motor-on-your-code-cycle) [integration](http://toddway.com/post/165735557485/continuous-integration-for-firebase-cloud-code), which is essentially an automated checklist verified by a machine. CI provides a level of safety, consistency, and efficiency that's hard to match in any other way.  The problem is some details are prohibitively hard to automate and too important to ignore.  Our current solution for this is to manually review all code before it "takes flight".  We use pull requests for this.  What we've been missing, tho, is a preflight checklist.

Gawande sees ineptitude (not making use of what we already know) as a greater problem than ignorance (what we don't know).  So our development group did a retrospective on code reviews.  We made a list of *what we already know* we're looking for when reviewing code.  The things we don't want to forget about in the future.   Based on input from that discussion and this handy [checklist for creating checklists](http://www.projectcheck.org/uploads/1/0/9/0/1090835/checklist_for_checklists_final_10.3.pdf), I started one:

## A checklist for reviewing code
	Integration  
	- [ ] Will merging this code create source conflicts?  
	- [ ] Is there a clear and concise description of the changes?
	- [ ] Did all automated checks (build, test, lint) run and pass?  
	- [ ] Are there supporting metrics or reports (e.g. test coverage, fitness functions) that measure the impact?
	- [ ] Are there obvious logic errors or incorrect behaviors that might break the software?

	Readability
	- [ ] Is the code self-documenting? Do we need secondary sources to understand it?  
	- [ ] Do the names of folders, objects, functions, and variables intuitively represent their responsibilities?  
	- [ ] Could comments be replaced by descriptive functions?  
	- [ ] Is there an excessively long object, method, function, pull request, parameter list, or property list? Would decomposing make it better? .  

	Anti-patterns
	- [ ] Does the code introduce any of the following anti-patterns?
	- [ ] Sequential coupling - a class that requires its methods to be called in order  
	- [ ] Circular dependency - mutual dependencies between objects or software modules  
	- [ ] Shotgun surgery - a change needs to be applied to multiple classes at the same time  
	- [ ] Magic numbers - unexplained numbers in algorithms  
	- [ ] Hard code - embedding assumptions about the environment in the implementation  
	- [ ] Error hiding - catching an error and doing nothing or showing a meaningless message  
	- [ ] Feature envy - a class that uses methods of another class excessively  
	- [ ] Duplicate code - identical or very similar code exists in more than one location.  
	- [ ] Boat anchor - retaining a part of a system that no longer has any use  
	- [ ] Cyclomatic complexity - a function contains too many branches or loops 
	- [ ] Famous volatility - a class or module that many others depend on and is likely to change 
	  
	Design principles
	- [ ] Does the code align with the following principles?
	- [ ] Single Responsibility - an object should have only one reason to change  
	- [ ] Open/Closed - objects should be open for extension, closed for modification  
	- [ ] Liskov Substitution - subtypes should not alter the correctness of code that depends on a supertype  
	- [ ] Interface Segregation - many client specific interfaces are better than one general purpose interface  
	- [ ] Dependency Inversion - dependencies should run in the direction of abstraction; high level policy should be immune to low level details

	Last updated: 8/15/2018

	Note: This is not a checklist for *approving* or *merging* code, it is a checklist for *reviewing* code.  It's a list of questions a reviewer should ask themselves as they review.