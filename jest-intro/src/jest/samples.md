# Samples

## Expressiveness

Here is a very expressive sample for a date/time validation function.

Using such detailed samples allows us to find the exact places and edge cases that a service might fail if adjusted.

In reality, I'd probably use an [`it.each()`](./repeated_tests.md) expression to write this, but it gives us a very well-described function anyhow.

```typescript
describe('validateDateTimeString', () => {
    describe('Date strings', () => {
        it('Accepts a well formatted string', () => {
            expect(validateDateTimeString('01/01/2021', 'dd/MM/yyyy')).toBeTruthy();
        });

        it("Rejects a string that requires leading zeroes, but doesn't supply them", () => {
            expect(validateDateTimeString('1/1/2021', 'dd/MM/yyyy')).toBeFalsy();
        });

        it('Rejects a date where the year position is not where it is expected', () => {
            expect(validateDateTimeString('2021/01/01', 'dd/MM/yyyy')).toBeFalsy();
        });

        it('Rejects a date where the day value is larger than can be in any month', () => {
            expect(validateDateTimeString('90/01/2021', 'dd/MM/yyyy')).toBeFalsy();
        });

        it('Rejects a date where the month value is larger than 12', () => {
            expect(validateDateTimeString('01/90/2021', 'dd/MM/yyyy')).toBeFalsy();
        });

        it('Rejects a date where the year value is longer than the required four digits', () => {
            expect(validateDateTimeString('01/01/12021', 'dd/MM/yyyy')).toBeFalsy();
        });

        it('Rejects a date where the day value (40) is larger than can be for a given month', () => {
            expect(validateDateTimeString('40/08/2021', 'dd/MM/yyyy')).toBeFalsy();
        });

        it('Rejects a date where the day value is in the months column (ie - American formatting)', () => {
            expect(validateDateTimeString('04/30/2021', 'dd/MM/yyyy')).toBeFalsy();
        });

        it('Accepts a date where the day value (30) is within the given month (April)', () => {
            expect(validateDateTimeString('30/04/2021', 'dd/MM/yyyy')).toBeTruthy();
        });

        it('Rejects a date where the day value (31) is larger than can be for a given month (April)', () => {
            expect(validateDateTimeString('31/04/2021', 'dd/MM/yyyy')).toBeFalsy();
        });

        it('Accepts a date where the day value (31) is within the given month (January)', () => {
            expect(validateDateTimeString('31/01/2021', 'dd/MM/yyyy')).toBeTruthy();
        });

        it('Rejects a date where the day value (32) is larger than can be for a given month (January)', () => {
            expect(validateDateTimeString('32/01/2021', 'dd/MM/yyyy')).toBeFalsy();
        });

        it('Accepts a date where the day value (28) exists for the given February (2022)', () => {
            expect(validateDateTimeString('28/02/2022', 'dd/MM/yyyy')).toBeTruthy();
        });

        it('Rejects a date where the day value (29) does not exist for the given February (2022)', () => {
            expect(validateDateTimeString('29/02/2022', 'dd/MM/yyyy')).toBeFalsy();
        });

        it('Accepts a date where the day value (29) exists for the given February (2024)', () => {
            expect(validateDateTimeString('29/02/2024', 'dd/MM/yyyy')).toBeTruthy();
        });

        it('Rejects a date where the day value (30) does not exist for the given February (2024)', () => {
            expect(validateDateTimeString('30/02/2024', 'dd/MM/yyyy')).toBeFalsy();
        });

        it('Accepts a date where the day value (28) exists for the given February (2100)', () => {
            expect(validateDateTimeString('28/02/2100', 'dd/MM/yyyy')).toBeTruthy();
        });

        it('Rejects a date where the day value (29) does not exist for the given February (2100)', () => {
            expect(validateDateTimeString('29/02/2100', 'dd/MM/yyyy')).toBeFalsy();
        });

        it('Rejects a date that is given in English instead of the required date string', () => {
            expect(validateDateTimeString('1st of January 2021', 'dd/MM/yyyy')).toBeFalsy();
        });
    });

    describe('Time strings', () => {
        it('Accepts a time string that is formatted correctly for the given format', () => {
            expect(validateDateTimeString('01:01:01', 'HH:mm:ss')).toBeTruthy();
        });

        it('Rejects a time with an hour value that is too high', () => {
            expect(validateDateTimeString('60:01:01', 'HH:mm:ss')).toBeFalsy();
        });

        it('Rejects a time with a minute value that is too high', () => {
            expect(validateDateTimeString('01:90:01', 'HH:mm:ss')).toBeFalsy();
        });

        it('Rejects a time with a second value that is too high', () => {
            expect(validateDateTimeString('01:01:90', 'HH:mm:ss')).toBeFalsy();
        });

        it('Rejects a time with a value written in english, instead of the required date format', () => {
            expect(validateDateTimeString('midnight', 'HH:mm:ss')).toBeFalsy();
        });
    });
});


```