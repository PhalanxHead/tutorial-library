# Mocking AWS Services with `aws-sdk-mock`

`aws-sdk-mock` gives the user a little more control over mocking the Aws SDK. Specifically, it overrides specific functions of the SDK with your own implementation, preserving the type contract.

  

Note: \`aws-sdk-mock\` is built for the 'v2' version of the \`aws-sdk\`.  
For v3 (ie the one that uses \`@aws-sdk/client-<service>\`), use [https://github.com/m-radzikowski/aws-sdk-client-mock](https://github.com/m-radzikowski/aws-sdk-client-mock)

  

The standard pattern for mocking with `aws-sdk-mock` and `jest` looks like so:

```typescript
import AWSMock from 'aws-sdk-mock';
import AWS from 'aws-sdk';
import { getFile } from 'app/service/s3-service'; // This is our wrapper service for S3

import type { GetObjectOutput, GetObjectRequest } from 'aws-sdk/clients/s3';

describe('s3-service', () => {
    describe('getFile', () => {
        // Create a Jest Mock Function that we can query about calls later.
        let mockGetObject: jest.Mock<GetObjectOutput, [req: GetObjectRequest]>;
        beforeAll(() => {
            // Write an implementation for our Jest Mock
            mockGetObject = jest.fn((req: GetObjectRequest): GetObjectOutput => {
                return { Body: 'Test Body', VersionId: 'V1' };
            });


            // Note that we need to set the AWS SDK instance before we try to mock it.
            AWSMock.setSDKInstance(AWS);
            // Overwrite S3.getObject() with AWSMock. 
            // Note that we use the `callback()` function, leaving the Error side undefined.
            AWSMock.mock('S3', 'getObject', (params, callback) => {
                callback(undefined, mockGetObject(params));
            });
        });

        afterAll(() => {
            // Restore S3's normal functionality
            AWSMock.restore('S3', 'getObject');
            // Clear out the Jest Mock as well, just to be safe
            jest.resetAllMocks();
        });

        afterEach(() => {
            // Clear data from the mock, ie how many times it was called, etc
            mockGetObject.mockClear();
        });

        // Test that our wrapper is returning what we told it to from the mock
        it('Returns the requested file if the file exists', async () => {
            const awsResp = await getFile('testBucket', 'testFolder/testFile.txt');

            expect(awsResp).toEqual({ Body: 'Test Body', VersionId: 'V1' });
        });

        // Test that we aren't calling S3 more than we have to
        it('Calls the getObject function once per request', async () => {
            await getFile('testBucket', 'testFolder/testFile.txt');

            expect(mockGetObject).toHaveBeenCalledTimes(1);
        });
    });
});


```

  

This pattern allows us to modify the return of an AWS SDK function individually, but also use a standard return in most cases.

  

Note that the AWS SDK class and method are described by strings, which are equivalent to the name of the function you are overriding in your code. `aws-sdk-mock` should warn you if you are overriding it incorrectly.
