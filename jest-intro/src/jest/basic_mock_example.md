# A Mock/Stub Example

Why would you want to mock or stub a function? In essence it comes down to the structure of your unit test.

Typically, each test should be looking to verify one specific piece of behaviour, eg: `The SMS Service sends one SMS to each unique number in the recipient list` .

This description implies 2 things:

*   The service is responsible for de-duplicating the recipient list
*   This service needs to call its SMS Provider once for each list item after de-duplication.

Realistically, we don't care who the SMS provider is, or how they are told to send an SMS, those can be handled downstream or in integration testing.

We probably don't want to be sending real SMSes out in this case though! So let's mock/stub the SMS Provider.

  

We may or may not care, in this tests case, what the response from the SMS Provider call is. If we do, we would stub the function to return some reasonable response, and if we don't, a mock is sufficient.

  

It can also be used to stand in for long running compute functions that we don't necessarily want to run every time we run `yarn test` .

  

For example, if we have a file like this:

```typescript
// /app/service/send-sms.ts
import { sendSms } from 'my_sms_library';

/**
 * Returns true when we successfully send an sms message
 */
export function sendSmsFromMe(destNumber: string, message: string): boolean {
  // Maybe do some validation on the destinationNumber

  try {
    const smsResponse = sendSms({
      sendingNumber: 'myNumber',
      destinationNumber: destNumber,
      message: message,
      apiKey: '<some precofigured thing>'
    });

    if(smsResponse.code === 200) {
      return true;
    }
  } catch { 
    return false; 
  }
  return false;
}
```

  

We may want to pretend to call `sendSms` instead of actually calling it (so we don't send a real sms out or spam the sms provider with junk).

However, Good Unit Testing principles dictate that we should make sure that `sendSmsFromMe` returns the correct response when `sendSms` gives us an error or something, so let's just pretend we own `sendSms` instead.

  

Below is the code for the minimal mock of the function where we tell it what we expect (I haven't tested this though whoops):

```typescript
// /test/service/send-sms.test.ts
import { sendSms } from 'my_sms_library';
import { sendSmsFromMe } from '/app/service/send-sms.ts';

jest.mock('my_sms_library');
// You can also specify the mock type, but I haven't here.
const mockedSendSms = sendSms as jest.Mock;

describe('send-sms', () => {
  describe('sendSmsFromMe', () => {

    afterEach(() => {
      // Remove any lingering metadata about the mock
      mockedSendSms.clearMock();
    });

    it('Returns true when the service returns a 200 code', () => {
      mockedSendSms.mockResolvedValue({ code: 200, messageId: '123445315' });
      expect(sendSmsFromMe('<my number>', 'Hello stranger!')).toBe(true);
    });

    it('Returns false when the service returns something other than a 200 code', () => {
      mockedSendSms.mockResolvedValue({ code: 400, error: `That didn't work!` });
      expect(sendSmsFromMe('<my number>', 'Hello stranger!')).toBe(false);
    });

    it('Returns false when the service throws an error', () => {
      mockedSendSms.mockImplementation(() => {throw new Error(`Hey! I don't like that!`)});
      expect(sendSmsFromMe('<my number>', 'Hello stranger!')).toBe(false);
    });
  });
});
```

We can also interrogate the mock to see how many times it was called, and with what arguments.

It'd not be good for say, an SMS Api to accidentally get called multiple times to send 1 SMS, so this can be helpful to test.

  

```typescript
// /test/service/send-sms.test.ts
import { sendSms } from 'my_sms_library';
import { sendSmsFromMe } from '/app/service/send-sms.ts';

jest.mock('my_sms_library');
// You can also specify the mock type, but I haven't here.
const mockedSendSms = sendSms as jest.Mock;

describe('send-sms', () => {
  describe('sendSmsFromMe', () => {

    afterEach(() => {
      // Remove any lingering metadata about the mock
      mockedSendSms.clearMock();
    });

    it('Returns true when the service returns a 200 code', () => {
      mockedSendSms.mockResolvedValue({ code: 200, messageId: '123445315' });
      expect(sendSmsFromMe('<my number>', 'Hello stranger!')).toBe(true);
    });

    it('Only calls the Sms Library once per sms request', () => {
      mockedSendSms.mockResolvedValue({ code: 200, messageId: '123445315' });
      sendSmsFromMe('<my number>', 'Hello stranger!');
      expect(mockedSendSms).toBeCalledTimes(1);
    });

    // You'd probably pre-configure these in some kind of config file, and you might keep a dummy copy in your repo for this particular test.
    it('Adds the api key and my number as the source!', () => {
      mockedSendSms.mockResolvedValue({ code: 200, messageId: '123445315' });
      sendSmsFromMe('<my number>', 'Hello stranger!');
      expect(mockedSendSms).toBeCalledWith({
        sendingNumber: 'myNumber',
        destinationNumber: '<my number>',
        message: 'Hello stranger!',
        apiKey: '<some precofigured thing>'
      });
    });

    // ... other tests redacted
  });
});
```

  

Don't forget you can use the scope of your containing block to reduce the number of strings you have to retype, or the number of similar mocks you need to set up!

  