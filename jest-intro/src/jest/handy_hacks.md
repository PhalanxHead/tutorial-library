# Hacks

## Boolean logic for Expect statements

We can write a fairly comprehensive test by using multiple `expect` statements in one test. ie:

```typescript
it('Does 2 things', () => {
  expect(thingOne).toBeTruthy();
  expect(thingTwo).toBeTruthy();
});
```

This gives us the equivalent of a logical AND statement - if anything fails, the whole test fails.

Jest, however, doesn't support logical OR on first inspection, ie:

```typescript
it('Does one of 2 things', () => {
  expect(result).toBe(thingOne jest.or thingTwo);
});
```

This makes sense somewhat, a unit really shouldn't have 2 equally valid outputs for 1 input.

  

However occasionally we might test something where it is helpful - like testing a mock where the call order doesn't matter.

  

In this case we can wrap the `expect` in a `try/catch` , and it will take either option. Example:

  

```typescript
it('Calls the mock service with one of 2 arg sets', () => {
  try{
    expect(mockService).toBeCalledWith({ argA: 'myArgA', argB: 'myArgB' });
  } catch {
    expect(mockService).toBeCalledWith({ argA: 'yourArgA', argB: 'yourArgB' });
  }
});
```

In this case, only 1 of the expect statements is required for the test to pass.

  