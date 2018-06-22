# Working Effectively with Legacy Code

Notes by Jeremy W. Sherman, October 2013, based on:

Feathers, Michael. _Working Effectively with Legacy Code_. Sixth printing, July 2007.

Foreword:
- Software systems degrade into a mess.
- Requirements ALWAYS change.
  - Your goal as a software developer: Create designs that tolerate change.
  - Tools: Principles, patterns, practices => prevent bit rot
- Bit rot still happens - entropy
  - Feathers book: Techniques to *reverse bit rot* – require effort to apply

Preface:
- "They're writing legacy code"
  - Legacy Code: "Legacy code is simply code without tests." (xvi)
- Book: Tools to "confidently make changes in any code base"
- Steps to making changes:
  - Understand
  - Harness to tests
  - Refactor
  - Add features
- "Extreme Programming is less a way to develop software than it is a way to make a well-jelled work team that just happens to deliver great software every two weeks." (xviii)
- Examples:
  - All seen IRL. Where you see an ellipsis, imagine 5000 more lines of ugly.
  - Java because it's common in enterprise.
  - C++ because it has special challenges.
  - C because it exemplifies legacy procedural code.

Introduction:
- Main part is structured as a FAQ (Part II, Changing Software).
  - Preceded by a part the explains fundamental background used to allow us to safely change software (Part I, The Mechanics of Change).
  - Followed by a catalog of useful refactorings (Part III, Dependency-Breaking Techniques).

Chapter 1: Changing Software
- Behavior is the heart of software
- Most of the time, seeking to preserve existing behavior
- Change means risk
  - Mitigate by:
    - Defining scope of change
    - Having objective criteria for success
    - Ensuring existing behavior was preserved

Chapter 2: Working with Feedback
- Regression testing is testing to detect change, not to show correctness (that's what QA does)
- Tests act as a software vise (p. 10): clamp down known good behavior, detect any unintended changes to it – gives us control over the work

Unit Testing:
- Regression testing useful but often too high-level; we want *unit tests*: small, localized tests.
- "Unit" - function in procedural code, class in object-oriented code
- "Test harness" - whatever test code we use to exercise our code
- Unit tests beat large integration tests in:
  - Error localization - where did the error happen - far smaller scope for units
  - Execution time - more focused tests => faster tests
  - Coverage - hard to exercise new code with high-level tests alone

Aside (p. 13): "I'll have big tests and just run them often." BEEP. No you won't, because they'll be slow. Integration tests are no substitute for unit tests.

- Key to unit tests is SPEED: >= 1/10 of a second is TOO LONG
  - Doesn't scale to large projects with many classes and consequently many tests.
- Disqualifiers (p. 14):
  - Talking to a database
  - Hitting the network
  - Hitting the disk
  - Requiring environment or config file twiddling before they can be run.

Test Coverings:
- Key to working with legacy code is breaking dependencies so that changes can be made more easily and under test. (p. 16)
- Legacy Code Dilemma: Need tests to change code; need to change code to add tests.
  - Solution: Very small, very conservative refactorings that can be safely made prior to tests.
  - Use these to nudge the system into a testable state.
- Breaking dependencies is worth introducing some ugliness. Think of it as a scar you can heal later once you've tests in place. (p. 18)

Legacy Code Change Algorithm:
- Identify change points
- Find test points
- Break dependencies
- Write tests
- Make changes and refactor

Notice that the LAST thing you do is write new code. Most of your work with legacy code is understanding and carefully selecting where to make changes so as to minimize risk and minimize the amount of new test code you have to write to make your target change safely.

Also note that last half of the last step: REFACTOR. This is often overlooked, both here and in the Red-Green-Refactor (too often: Red-Green-Repeat) TDD cycle, but it's crucial.

Chapter 3: Sensing and Separation
- We break dependencies so we can write unit tests.
- We break dependencies with 2 proximate goals:
  - SENSING: Provide access to values produced by a computation inside a class or method where we couldn’t observe them for testing.
  - SEPARATION: Make a test target independent enough that we can actually stand it up in a test harness.

Faking Collaborators:
- “Fake object”: An object that impersonates some collaborator of your class when it is being tested. (p. 23)
- Two sides:
  - API for the system under test. This is what we’re faking out.
  - API for the test code. These are hooks to look inside and examine what the SUT is doing to the fake, so we can make assertions about it.
- “Mock object”:  Fakes that perform assertions internally. (p. 27)
  - Mock object frameworks are powerful, but not always available. Simple fake objects suffice in most cases.
 
A fake object is often just a FakeWhatever class that inherits from Whatever, overrides the methods we’re faking out, and adds some API for the test to use.

Chapter 4: The Seam Model
- Good design vs. good *testable* design – they are not the same.
- “Seam”: A place where you can alter behavior in your program without editing in that place. (p. 31 and again on p. 36)
- “The source code should be the same in both production and test” (p. 36), but we need different behavior in tests vs. production; seams let us accomplish both aims.
- “Enabling point”: The place where you can make the decision to one behavior or another.
Seam Types:
- Preprocessing: use #define to rewrite calls, or #include + #if TEST and non-test code; enabling point is the preprocessor #define TEST (0) or (1)
- Link: rewrite seam via classpath (enabling point); rewrite seam by linking to test version of library rather than production version
  - Warning: Take pains to make sure the difference between test and production environments is obvious when using link seams, because it’s easy for that to get buried in a build script somewhere. (p. 40)
- Object: often dependency injection or subclass-and-override
  - “pretty much the most useful seams available in object-oriented programming languages” (p. 40)

Chapter 14: Dependencies on Libraries Are Killing Me
- Avoid littering direct calls to library classes in your code. You might think that you’ll never change them, but that can become a self-fulfilling prophecy!
  - Coupling tightly to a third-party library => they can extort the daylights out of you, or burn you when they drop support for your library/platform/toolchain.
- “Every hard-coded use of a library class is a place where you could have had a seam” (p. 197).
  - Final/sealed classes just make testing hard. You have to add a facade to paper over the damage. Jerks.
- “once dilemma”: If the library assumes that there is going to be only one instance of a class in the system, it can make the use of fake objects difficult.
  - Specifically talking about where you can’t new-up a new instance and swap in your test double for their singleton.
  - Advice: Avoid singletons in your own code. There’s no reason to do this to yourself.
-  “restricted override dilemma”: Making a method non-virtual makes it so you can’t override it in a fake object.
  - Advice: Make all your member functions virtual. There’s no reason to do this to yourself.
  - “Sometimes using a coding convention is just as good as using a restrictive language feature. Think about what your tests need” (p. 198).
