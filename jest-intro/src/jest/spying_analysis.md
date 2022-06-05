# Spying on functions with Jest

Okay, so we can see the obvious use case for mocking - we don't want to call an AWS Service for example, but maybe we are testing something that handles some data it might return.

But when would we use spying? And how do we call it in Jest?

  

Generally, spying is used when we don't want to change the implementation of a class, but we do care about how it's being used.

  

Here's some generic examples where you'd maybe want to use a spy over a mock:

*   I need to test my http client service is only sending one request when I call it, and is not meaningfully modifying the responses. I'd use a Spy + a tool like Nock to create generic test cases.

When testing the consumers of this service, I will write a stub for this service.

  

*   I have some publisher/subscriber model, and I really need to make sure the subscriber is receiving notifications in specific circumstances (ie: I have a checkbox checked).

I don't want to mock this subscriber particularly, but I need to verify that the subscription and unsubscription is actually working as I intended.
