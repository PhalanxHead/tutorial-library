# Mocking Singletons with Jest

Singletons are a common design pattern in a reasonable amount of software. The typical case is that you have some configuration data that you want to use in your service that you don't want to retrieve or parse each time the service is created.

  

Unfortunately, since singletons give you a weird call pattern, mocking their functions is a bit of a pain.

  

One solution is to refactor the singleton service to be provided via [dependency injection](https://app.clickup.com/6943244/v/dc/6kwgc-1766/6kwgc-72782), but this does add some amount of structural overhead to your application, may require learning a framework, and otherwise requires justifying the work to actually implement.

  

We can, however, mock the singleton functions using Jest Spies.

  

Wait! I just said that spies don't change their target's behaviour!

Well... They can in Jest! Probably because they can in other mocking frameworks, and Jest thinks it wants to be the testing framework to end all the rest.

It's a good thing too, since the way that regular jest mocking works isn't really conducive to dealing with class instances.

  

Take for example the following service and consumer, and let's say we want to test the consumer works properly.

  

```typescript
// src/service/db-service.ts
export class DbService {
	private static instance: DbService;
	public static getInstance(): DbService {
		if (this.instance == null) {
			this.instance = new DbService();
		}
		return this.instance;
	}

	private connectionString: string;

	private constructor() {
		if (!process.env.DB_CONNECTION_STRING) {
			throw new Error('process.env.DB_CONNECTION_STRING not set');
		}
		this.connectionString = process.env.DB_CONNECTION_STRING;
	}

    /** Retrieves the record with the given ID from the database */
	async getRecord(recordId: string): Promise<{ id: string; value: string }> {
		// Actual implementation removed because we don't care, let's return some random string for now.
		return { id: recordId, value: 'Hello World!' };
	}
}
```

  

```typescript
// src/service/email-service.ts

/** Retrieves a given record from the DB and renders it as HTML */
export async function renderRecordAsHtml(recordId: string): Promise<string> {
	try {
		const record = await DbService.getInstance().getRecord(recordId);
		return `<!Doctype html><html><body><h1>${record.id}</h1><p>${record.value}</p></body></html>`;
	} catch (e) {
		throw new Error('Something went horribly wrong');
	}
}
```

  

```typescript
// test/service/email-service.ts

describe('renderRecordAsHtml', () => {
    const mockRecordValue1 = 'Our own special record value!';
	let dbServiceInstance: DbService;
	let dbServiceSpy: jest.SpyInstance;

	beforeAll(() => {
		process.env.DB_CONNECTION_STRING = 'blah!';
		// Get the service *before* running the suite, as we need to use the same class instance to mock it.
		dbServiceInstance = DbService.getInstance();

		// Now let's mock it
		dbServiceSpy = jest.spyOn(dbServiceInstance, 'getRecord');
		dbServiceSpy.mockImplementation((recordId: string) => {
			return { id: recordId, value: mockRecordValue1 };
		});
	});

	afterEach(() => {
		dbServiceSpy.mockClear();
	});

	afterAll(() => {
		delete process.env.DB_CONNECTION_STRING;
		jest.resetAllMocks();
	});

	// We use our default (set in beforeAll()) if we don't override it
	it('Inserts the record ID into a H1 tag', async () => {
		const recordAsHtml = await renderRecordAsHtml('314');
		expect(recordAsHtml).toEqual(expect.stringContaining(`<h1>314</h1>`));
	});

	it('Inserts the value inside a paragraph tag', async () => {
		const recordAsHtml = await renderRecordAsHtml('314');
		expect(recordAsHtml).toEqual(expect.stringContaining(`<p>${mockRecordValue1}</p>`));
	});

	// Or we can override it for this specific test set!
	it('Inserts the record ID into a H1 tag after overriding the implementation', async () => {
		const mockIdOnce = 'This is a fake ID!';
		const mockRecordOnce = 'This is our own record :p';
		dbServiceSpy.mockImplementationOnce(() => {
			return { id: mockIdOnce, value: mockRecordOnce };
		});

		const recordAsHtml = await renderRecordAsHtml('314');
		expect(recordAsHtml).toEqual(expect.stringContaining(`<h1>${mockIdOnce}</h1>`));
	});

	// But this one still uses the default implementation!
	it('Inserts the value inside a paragraph tag', async () => {
		const recordAsHtml = await renderRecordAsHtml('314');
		expect(recordAsHtml).toEqual(expect.stringContaining(`<p>${mockRecordValue1}</p>`));
	});

	// We can even make it throw an error!
	it('Throws ~Something went horribly wrong~ if the record is not found', async () => {
		dbServiceSpy.mockImplementationOnce(() => {
			throw new Error('Heck!');
		});

		try {
			const recordAsHtml = await renderRecordAsHtml('314');
			fail('Expected to throw error "Something went horribly wrong", but did not');
		} catch (e) {
			expect(e).not.toBeNull();
			expect(e).toEqual(new Error('Something went horribly wrong'));
			expect(e).not.toEqual(new Error('Heck!'));
		}
	});
});
```

The above will correctly mock the implementation of the `DbService`, but we still need to know the implementation details of `DbService.getInstance()`.

  

In particular, if you are testing multiple functions inside `email-service.ts` , you will need to allow a singleton reset so you aren't sharing state between tests, or structure your tests such that all the mocking is done before all the suites run. This is pretty annoying IMO.
