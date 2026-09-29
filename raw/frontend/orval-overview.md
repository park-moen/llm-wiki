# Orval: Overview

> Source: https://orval.dev/docs/
> Collected: 2026-09-29
> Published: Unknown

Orval generates type-safe TypeScript clients from any valid OpenAPI v3 or Swagger v2 specification, either in YAML or JSON formats.

## Why Orval?

Developers often use OpenAPI/Swagger as a contract between frontend and backend. Before Orval, tools like Swagger Editor or Swagger Codegen did the job. But that's not enough for modern development needs. Enter Orval.

### Main Goals

- Generate TypeScript models — Full type safety for your API responses and requests
- Generate HTTP request functions — Ready-to-use functions with proper typing
- Generate mocks with MSW — Test your frontend without a backend

## Supported Clients

The default generated clients use Fetch API. You can configure any client (e.g. axios) depending on your project needs.

React Query, Vue Query, Svelte Query, Solid Query, Angular, SWR.
