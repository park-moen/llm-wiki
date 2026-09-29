# Orval: Quick Start

> Source: https://orval.dev/docs/quick-start/
> Collected: 2026-09-29
> Published: Unknown

## Without a Configuration File

The simplest way to use Orval is directly from the command line:

```sh
orval --input ./petstore.yaml --output ./src/petstore.ts
```

- `--input` accepts any YAML or JSON-formatted OpenAPI specification
- `--output` specifies the target path for the generated files

## With a Configuration File

For more control, create a configuration file. Orval will automatically look for `orval.config.js`, `orval.config.ts`, or `orval.config.mjs` in your project root.

```sh
orval --config ./orval.config.js
# or simply
orval
```

### Example Configuration

```ts
import { defineConfig } from 'orval';

export default defineConfig({
  petstore: {
    input: './petstore.yaml',
    output: './src/petstore.ts',
  },
});
```

## Generate React Query Hooks

Want to generate React Query hooks instead? Just set the `client` option:

```ts
import { defineConfig } from 'orval';

export default defineConfig({
  petstore: {
    input: './petstore.yaml',
    output: {
      target: './src/api/petstore.ts',
      schemas: './src/api/model',
      client: 'react-query',
    },
  },
});
```
