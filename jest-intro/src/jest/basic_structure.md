# Basic Structure of Jest Tests

Jest is a test and mocking framework built in and for NodeJS. Typescript compatibility has been built over the top of it, but a lot of the legacy documentation has been written for older Js in mind first.

  

The basic intent, as with all unit testing, is to take some public element of your codebase (a unit if you will), and ensure that it works the way you expect under all input conditions. This can help us detect the cause of bugs early while building, and when a change has created a problem later on.

  

Jest has a series of different key functions we can use to organise and run our tests. Here's the main few:

  

```typescript
describe('group description', () => {});
// Describe is used to group logically similar test cases. You can nest these as many times as needed. Expect() statements do not go into describe() blocks directly.

it('test description', () => {});
test('test description', () => {});
// It and Test are aliases of the same thing. No preference for either, apart from how you phrase your test descriptions. 
// it('Returns a positive result when adding 2 positive numbers', () => {})
// is preferred over
// test('If it returns a positibe result when adding 2 positive numbers', () => {});
// As it's more obvious what the test is expecting when it fails.

expect();
// expect is the assertion clause that Jest uses. It has 2 basic forms:
// 1: Matching a result with an expectation:
expect(myFunction('Hello', 'World!')).toBe('Hello World!');
// 2: Checking for an error
expect(() => { myFunction('Hellow', 'World') }).not.toThowError();
```

  

When put together, normal, synchronous testing might have a structure like this:

```typescript
describe('testModule/Class', () => {
  describe('public function being tested', () => {
    it('specific behaviour being tested 1 - Eg "Adding 2 positive integers gives the positive sum"', () => {
      expect(add(2,2)).toBe(4);
    });

    it('specific behaviour being tested 2 - Eg "Adding a positive and negative integer gives the signed difference between their absolute values"', () => {
      expect(add(2,-2)).toBe(0);
    });

    it('specific behaviour being tested 3 - Eg "Adding two negaive integers gives the negative sum"', () => {
      expect(add(-2, -2)).toBe(-4);
    });
  });
});
```

Generally speaking, less assertions per-test (ie, per `it()` statement) will give more reliable results. Jest has been known to ignore failing `expect()` statements if several (>5 maybe?) are stacked.

Writing good unit tests with descriptive labels also has the advantage of acting as documentation for a specific service. Comments and requirements docs may fail to be updated at some point, but tests must pass to be deployed, so it is helpful to write more, smaller tests.

  

A more explicit form of Arrange/Act/Assert might look like so:

```typescript
describe('testModule/Class', () => {
  describe('public function being tested', () => {
    it('specific behaviour being tested 1 - Eg "Adding 2 positive integers gives the positive sum"', () => {
      const inputDataA = 2;
      const inputDataB = 2;
      const expectedOutputData = 4;

      const actualResult = add(inputDataA, inputDataB);

      expect(actualResult).toBe(expectedOutputData);

    });
  });
});
```

Obviously this is a fairly contrived example, but your data may be a lot more complex - ie data produced by faker or a very long string.

  