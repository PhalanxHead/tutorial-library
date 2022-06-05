# Mocking AWS PromiseResult

Occasionally, there is the use case in the AWS SDK to inspect the raw `$response` object that AWS returns inside some SDK Result. This contains errors and the raw HTTP response object.

Unfortunately, doing this in application code means that you also need to mock it in your AWS Mocks.

Below is an example of this working correctly for KMS:

  

```typescript
// /test/mocks/aws-result.ts
import AWS from 'aws-sdk';
import { PromiseResult } from 'aws-sdk/lib/request';

/** This produces a valid AWS PromiseResult that a mock funtion could return */
export function awsSuccessPromiseResult<T>(resp: T): PromiseResult<T, AWS.AWSError> {
	return {
		...resp,
		$response: {
			data: resp,
			hasNextPage: () => {
				return false;
			},
			requestId: '',
			redirectCount: 0,
			retryCount: 0,
			nextPage: () => {},
			error: undefined,
			httpResponse: {
				body: '',
				headers: {},
				statusCode: 200,
				statusMessage: 'OK',
				streaming: false,
				createUnbufferedStream: () => {
					return {};
				},
			},
		},
	};
}

/** This produces a valid AWS Error PromiseResult response */
export function awsErrorPromiseResult(): PromiseResult<any, AWS.AWSError> {
	const otherError: AWS.AWSError = {
		code: 'ServiceUnavailable',
		message: 'need more kfc 21 piece buckets',
		name: 'ServiceUnavailable',
		time: new Date(),
	};
	return {
		$response: {
			data: undefined,
			hasNextPage: () => {
				return false;
			},
			requestId: '',
			redirectCount: 0,
			retryCount: 0,
			nextPage: () => {},
			error: otherError,
			httpResponse: {
				body: '',
				headers: {},
				statusCode: 200,
				statusMessage: 'OK',
				streaming: false,
				createUnbufferedStream: () => {
					return {};
				},
			},
		},
	};
}
```


```typescript
// /test/jwt-service.test.ts
import jwt from 'jsonwebtoken';
import AWSMock from 'aws-sdk-mock';
import AWS from 'aws-sdk';
import KMS from 'aws-sdk/clients/kms';
import { PromiseResult } from 'aws-sdk/lib/request';
import { awsSuccessPromiseResult } from './mocks/aws-results';

describe('JwtService', () => {
	let mockSign;
	beforeAll(() => {
		mockSign = jest.fn((req: KMS.SignRequest): PromiseResult<KMS.SignResponse, AWS.AWSError> => {
			const resp = {
				KeyId: req.KeyId,
				SigningAlgorithm: req.SigningAlgorithm,
				Signature: 'ThisSignatureVerifiesThatTheJwtIsToooootallyLegit',
			};
			return awsSuccessPromiseResult(resp);
		});

		// Overwriting KMS.sign()
		AWSMock.setSDKInstance(AWS);
		AWSMock.mock('KMS', 'sign', (params, callback) => {
			callback(undefined, mockSign(params));
		});
	});

	afterAll(() => {
		// Restore KMS
		AWSMock.restore('KMS', 'sign');
		jest.resetAllMocks();
	});

	describe('signJwt', () => {
		afterEach(() => {
			mockSign.mockClear();
		});

		it('Mocks the signature correctly', async () => {
			const response = await new KMS().sign({
			    Message: '{"recordA":"A very transgender value", ...<other JWT stuff>}',
			    KeyId: '99999',
			    SigningAlgorithm: 'RSASSA_PKCS1_V1_5_SHA_256',
			    MessageType: 'RAW',
		    })
		    .promise();
			console.log(response);
			console.log(JSON.stringify(jwt.decode(response)));
			expect(true).toBeTruthy();
		});
	});
});
```

  