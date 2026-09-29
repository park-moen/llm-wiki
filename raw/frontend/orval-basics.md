# Orval: Basics

> Source: https://orval.dev/docs/guides/basics/
> Collected: 2026-09-29
> Published: Unknown

Start by generating or defining an OpenAPI specification (example petstore.yaml).

Then create a file `orval.config.ts` at the root of your project.

## Basic Configuration

```ts
import { defineConfig } from 'orval';

export default defineConfig({
  petstore: {
    output: {
      mode: 'single',
      target: './src/petstore.ts',
      schemas: './src/model',
      client: 'react-query',
      httpClient: 'fetch',
      mock: true,
    },
    input: {
      target: './petstore.yaml',
    },
  },
});
```

`client` and `httpClient` control different parts of the generated output. The `client` selects the generated API style, while `httpClient` selects the request implementation used by compatible clients. For example, the configuration above generates React Query hooks backed by the Fetch API.
