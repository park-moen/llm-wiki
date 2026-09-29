# Orval: MSW

> Source: https://orval.dev/docs/guides/msw/
> Collected: 2026-09-29
> Published: Unknown

Generate MSW (Mock Service Worker) handlers from your OpenAPI specification to mock your API during development and testing.

For mock data factories without MSW request handlers, see the Faker guide.

## Configuration

Set the `mock` option to `true` (emits both MSW handlers and Faker factories), or scope it to MSW only via the generator entry:

```ts
import { defineConfig } from 'orval';

export default defineConfig({
  petstore: {
    output: {
      mode: 'single',
      target: './src/api/petstore.ts',
      schemas: './src/api/model',
      mock: {
        generators: [{ type: 'msw' }],
      },
    },
    input: {
      target: './petstore.yaml',
    },
  },
});
```

The MSW generator emits two types of functions per operation, plus an aggregator. Mock data (the `get<Op>ResponseMock` factories) is produced by the Faker generator — see the Faker guide for details on overriding values and formats.
