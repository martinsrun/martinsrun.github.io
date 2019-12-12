Structural testing:
  - Tests are dependent on structure of program, make sure all lines of source code are a part of some test case. Line coverage ensures all lines in your code is covered by atleast one test.
  - Line coverage may not be the best approach since number of lines are dependent on developer. Structural coverage can include metrics  such as statement coverage, the percentage of statements covered by a test, or instruction coverage, the percentage of bytecode instructions covered.
  - Branch coverage is similar to line coverage but counts percentage of branches covered and not just the lines. Branches are the code coming under each each control structure. Condition coverage ensures each sub conditional are evaluated to both true and false.
  - Path coverage is covering all possible paths the the program might take, but achieving this is not easy. Modified condition decision coverage (MC/DC) is testing only certain inputs to ensure all paths are covered. For each possible input, find all smallest subset of possible values such that affect output while all other inputs remain the same.


Unit testing
  - A unit is a separately testable software item. It can be a class, a module, an object etc, and span one or more classes.
System testing
  - Black-box testing. Test the entire system as a whole, with business use cases.
Integration Tests
  - Between unit and system tests, where you test the integration between two systems. If you have a db access in your app, testing it is an integration test.
https://martinfowler.com/articles/practical-test-pyramid.html

Mock Objects:
  - When your test unit is a business rule that depends on an external system, it's better to test your rule in isolation rather that having the actual integration in the test code.
  - Mock objects are simulations, usually of hard to test dependencies.  https://site.mockito.org/ https://dzone.com/refcardz/mockito?chapter=1
  - Code should be mock friendly. External objects should be a dependency and not hard declared in your class in order to be mocked.
  - Controllability: How easy it is to set up test units and provide inputs to invoke the desired behaviors on the system.
  - Observability: How easy it is to set up tests to control and verify if the system behaved as expected.
  - Design for testability - If your class is offering a discount on Christmas day, inject the current date rather than fetching it from the Date class, else you will have to wait until Christmas to actually test it.

Principals of testable systems:
  - Domain model should be separate from infrastructure code. Your business logic should be independent from the database or request handling system in order to test independently.
  - Ports & Adapters - When your business classes need something from the infrastructure class make it dependent on a port, which is an abstract idea of what the infrastructure class returns. The actual implementation for the port fetches from an Adaptor which talks to the infra system, but during testing the port can be mocked out.

Maintaining test code:
  - Properties of good tests:
    - F :Fast, minimize dependencies on slow things
    - I : Isolated
    - R : Repeatable, same results regardless of number of runs
    - S : Self-validating
    - T : Timely, should come along with development

  - Test smells exist. Things that make test suites hard to maintain or comprehend.    
    - Duplication, test methods having duplicate code.
    - Assertion roulette, Test fails but had to  understand why, assertion may be too complex or do too many things.
    - Slow tests, people don't want to run tests that take time to execute.
    - Resource optimism, expecting a resource is available while a test runs.
    - Test run war, multiple systems running the same tests make it fail. Can happen when depending on same db.
    - General fixture, the set-up part of the test is too large and too generic.
    - Indirect tests
    - Sensitive equality, assertions are too sensitive, any small change will cause it to break.

  - Flaky Tests, tests that pass sometime and fail sometime. It makes the test suite undependable. It could be due to external dependencies, shared resources, timeouts or tests that interact with each other.
