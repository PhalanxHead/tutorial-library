# Testing Services with Jest

## Mock vs Stub vs Spy

Most testing reference documents will discuss some combination of Mocking, Stubbing and Spying as a tool for testing services. However, terminology has become muddled as more and more digital ink has been spilled, and frameworks have been introduced.

Let's briefly define the above terms.

  

### Mocking

Mocking is when you replace the implementation of a function, service, or other coroutine with a no-op operation, and return null or 0. Often you will use this to remove some external dependency like a HTTP call, or a send to a logging framework. It can also be used to replace some expensive call with a no-op.

Technically speaking, mocks don't do any calculation or return any real implementation/data. They're the dumbest possible unit, and we use them to replace side-effects.

  

### Stubbing

Stubbing is like mocking, but your replacement function might have a tiny amount of brains to it, and might return actual, useful data. It simulates some behaviour.

This article will use Mock and Stub interchangeably, as `jest` doesn't draw any real distinction between them (apart from mocks are not given implementations by default).

  

### Spying

Spying, unlike mocking or stubbing, doesn't replace the default implementation of a piece of code (at least, unless you tell it to). Generally you would use spying to check that a method is being called with the right parameters, and the correct number of times.
