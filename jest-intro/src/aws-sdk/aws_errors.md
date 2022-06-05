# Mocking AWS Errors without `aws-sdk-mock`

When testing unit behaviour in the event of an error, it can be helpful to mock when AWS returns some kind of error. The `aws-sdk` typically throws an `AWSError` object when it fails (as a promise rejection).

Unfortunately, this data type is not something `jest` will typically handle in its `expect().toThrow()` matching hook.

This pattern is known to work if there is a wrapper typescript service around the AWS SDK service (in the below example as `/app/service/s3-service.ts` ), but could potentially work with `aws-sdk-mock` as well.

We can use the following pattern to mock an error response. This sample uses S3.

  

```typescript
// File: /app/service/s3-service.ts
import { S3 } from 'aws-sdk';

/**
 * Retrieves a file from an S3 Bucket as an S3 Object
 * @param Bucket The S3 bucket to fetch from
 * @param Key The name + path of the file to fetch
 * @returns The file as an S3 Object Promise
 */
export const getFile = async (Bucket: string, Key: string) =>
    new S3()
        .getObject({
            Bucket,
            Key,
        })
        .promise();


```

  

```typescript
// File: /app/service/s3-wrapper-service.ts
import * as s3 from './s3-service';
import type { PromiseResult } from 'aws-sdk/lib/request';
import type { AWSError, S3 } from 'aws-sdk';

/**
 * Fetches the contents of a file from an S3 bucket as a string. Returns an empty string if the file doesn't exist. Throws an {@link AWSError} on any other failure.
 * @param bucketName The name of the S3 bucket to read from
 * @param key The file name + path to read.
 * @returns The body of the file as a string. If the file doesn't exist, returns an empty string. If there is another error, throws it.
 */
export async function fetchFromS3(bucketName: string, key: string): Promise<string> {
    let file: PromiseResult<S3.GetObjectOutput, AWSError>;

    try {
        file = await s3.getFile(bucketName, key);
    } catch (e) {
        const error = e as AWSError;

        // If no files exists - just treat it as an empty file
        if (error.code === 'NoSuchKey') {
            return '';
        }
        throw e;
    }
    if (!file.Body) {
        // If the file doesn't exist - just treat it as an empty one
        return '';
    }
    return file.Body.toString();
}
```

  

```typescript
// File: /test/s3-wrapper-service.test.ts
import * as s3 from 'app/service/s3-service';
import { fetchFromS3 } from 'app/service/s3-wrapper-service';
import type { AWSError } from 'aws-sdk';

jest.mock('app/service/s3-service');

describe('fetchFromS3', () => {
    it('Returns an empty string if S3 returned `NoSuchKey` as an error', async () => {
        const noSuchKeyError: AWSError = {
            code: 'NoSuchKey',
            message: 'NoSuchKey error',
            name: 'No Such Key error or something',
            time: new Date(),
        };
        (s3.getFile as jest.Mock).mockRejectedValue(noSuchKeyError);
        const result = await fetchFromS3(process.env.DH_MIRROR_BUCKET!, 'testfile.csv');
        expect(result).toEqual('');
    });
    
    it("Throws S3 errors that aren't `NoSuchKey`", async () => {
      const otherError: AWSError = {
          code: 'ServiceUnavailable',
          message: 'need more kfc 21 piece buckets',
          name: 'ServiceUnavailable',
          time: new Date(),
      };
      (s3.getFile as jest.Mock).mockRejectedValue(otherError);
    
      try {
          await fetchFromS3('test-bucket', 'current.json');
          fail('Expected to throw but did not');
      } catch (e) {
          expect(s3.getFile).toBeCalledWith('test-bucket', 'current.json');
          expect(e).toBe(otherError);
      }
    });
});
```

Note the `try/fail/catch` block in the second `it()` statement.

  