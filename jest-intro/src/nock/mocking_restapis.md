# Mocking RestAPIs with Nock

[nock/nock: HTTP server mocking and expectations library for Node.js (github.com)](https://github.com/nock/nock)

Nock is a framework that allows the tester to mock restAPI responses, in HTTP format even.

This allows us to test our code against diverse responses, without the need for an integration server.

It can also let us get a little creative with our validation code, and ensure that we are handling unexpected responses correctly.

  

For example, the `axios` library provides async http calls to other services, but itself as a service can be hard to mock correctly, as its error handling can be somewhat subtle.

`nock` allows us to verify our code against what axios would actually do in a specific reaponse case (say, a 404 error), as opposed to us guessing based off their documents.

  

Nock automatically converts header keys in requests to lower case. In most cases, API gateways should also be case insensitive, but be sure to check this before relying on Nock to test your request headers.

For example, the following header arrangements should be equivalent on most HTTP servers:

```json
{
    "Authorization":"MyAuthKey", 
    "accept":"application/json", 
    "deviceId":"12345"
}

{
    "authorization":"MyAuthKey", 
    "accept":"application/json", 
    "deviceid":"12345"
}
```

However some specific server implementations may not be the case. Using Nock to test your header arrangements are cased correctly may not be a suitable test in this case.

  

The following describes a possible test pattern using Nock to control axios calls:

```typescript
import nock from 'nock';
import * as AuthService from '/app/service/authentication-service';
import { CustomError } from '/app/service/error';
import axios from 'axios';

// Make sure Axios will work with Nock
axios.defaults.adapter = require('axios/lib/adapters/http');

describe('authenticate', () => {
    const authDomainRoot = 'https://example.com';
    const authUrlPath = '/auth';

    afterEach(() => {
        // Don't forget to reset any Nock mocks after each test (since we are mocking per-test)
        nock.cleanAll();
    });

    it('Throws a specific error if the Auth API returns 400', async () => {
        const scope = nock(authDomainRoot).post(authUrlPath).reply(400, {});

        try {
            const response = await AuthService.authenticate('thisIsAnAuthCode', 'thisIsNotAJwt');
            console.log(response);
            fail('Expected to throw CustomError but did not');
        } catch (e) {
            if (!(e instanceof CustomError)) {
                fail('Expected to throw CustomError, but threw something else');
            } else {
                const specificCustomErr = new CustomError(400, '07', `Something went wrong, please try again`);
                expect(e.errorCode).toEqual(specificCustomErr.errorCode);
                expect(e.message).toEqual(specificCustomErr.message);
                expect(e.httpCode).toEqual(specificCustomErr.httpCode);
            }
        }
    });

    it('Throws a specific error if the Auth API returns 500', async () => {
        const scope = nock(authDomainRoot).post(authUrlPath).reply(500, {});

        try {
            const response = await AuthService.authenticate('thisIsAnAuthCode', 'thisIsNotAJwt');
            console.log(response);
            fail('Expected to throw CustomError but did not');
        } catch (e) {
            if (!(e instanceof CustomError)) {
                fail('Expected to throw CustomError, but threw something else');
            } else {
                const specificCustomErr = new CustomError(500, '10', `Something went wrong, please try again`);
                expect(e.errorCode).toEqual(specificCustomErr.errorCode);
                expect(e.message).toEqual(specificCustomErr.message);
                expect(e.httpCode).toEqual(specificCustomErr.httpCode);
            }
        }
    });

    it('Returns the same tokens provided by the Auth API', async () => {
        const mockAccessToken = 'ThisIsAnAccessToken';
        const mockRefreshToken = 'ThisIsARefreshToken';
        const scope = nock(authDomainRoot)
            .post(authUrlPath)
            .reply(200, { access_token: mockAccessToken, refresh_token: mockRefreshToken });

        const response = await AuthService.authenticate('thisIsAnAuthCode', 'thisIsNotAJwt');
        expect(response.accessToken).toEqual(mockAccessToken);
        expect(response.refreshToken).toEqual(mockRefreshToken);
    });

    it('The provided authCode is present in the body', async () => {
        const mockAccessToken = 'ThisIsAnAccessToken';
        const mockRefreshToken = 'ThisIsARefreshToken';
        const mockAuthCode = 'thisIsAnAuthCode';
        let caughtBody;
        const scope = nock(authDomainRoot)
            .post(authUrlPath)
            .reply(200, function (uri, reqBody) {
                caughtBody = reqBody;
                return { access_token: mockAccessToken, refresh_token: mockRefreshToken };
            });

        await AuthService.authenticate(mockAuthCode, 'thisIsNotAJwt');
        const bodyAsParams = new URLSearchParams(caughtBody);
        expect(bodyAsParams.get('code')).toEqual(mockAuthCode);
        expect(bodyAsParams.get('grant_type')).toEqual('authorization_code');
    });

    it('The provided clientAssertion is present in the body', async () => {
        const mockAccessToken = 'ThisIsAnAccessToken';
        const mockRefreshToken = 'ThisIsARefreshToken';
        const mockJwt = 'thisIsNotAJwt';
        let caughtBody;
        const scope = nock(authDomainRoot)
            .post(authUrlPath)
            .reply(200, function (uri, reqBody) {
                caughtBody = reqBody;
                return { access_token: mockAccessToken, refresh_token: mockRefreshToken };
            });

        await AuthService.authenticate('thisIsAnAuthCode', mockJwt);
        const bodyAsParams = new URLSearchParams(caughtBody);
        expect(bodyAsParams.get('client_assertion')).toEqual(mockJwt);
        expect(bodyAsParams.get('client_assertion_type')).toEqual(
            'urn:ietf:params:oauth:client-assertion-type:jwt-bearer'
        );
    });
});
```
