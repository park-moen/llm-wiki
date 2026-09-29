# Orval: Input Validation and External References

> Source: https://orval.dev/docs/reference/configuration/input/
> Collected: 2026-09-29
> Published: Unknown

## externalRefs

Control how external `$ref` targets (local files or remote URLs) are resolved.

By default, orval refuses to resolve any external `$ref` and prints a config snippet you can paste into your `parserOptions`.

Use `['*']` to allow all external refs (previous behavior). Orval will emit a warning listing external documents referenced by the top-level spec.

Security: External `$ref` values come from the spec being processed, which may be untrusted. Allowing all external refs (`['*']`) means orval will read arbitrary local files and fetch arbitrary URLs referenced by the spec. Prefer listing specific documents you trust.

## unsafeDisableValidation

Disable OpenAPI spec validation during code generation.

Type: `boolean` Default: `false`

Use at your own risk. Code generation from an invalid OpenAPI spec is not guaranteed to work and may break in minor updates. Bug reports with validation disabled will not be accepted.

When `true`, orval skips both spec-level validation (`@scalar/openapi-parser`) and the component-key check, and proceeds with code generation regardless of spec errors. Intended as an escape hatch for specs that use non-standard extensions (e.g. FastAPI's `itemSchema` on `text/event-stream` responses) which a compliant validator would otherwise reject. Prefer `override.transformer` — which runs before validation — when the spec can be repaired in-place.
