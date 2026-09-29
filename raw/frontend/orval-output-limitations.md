# Orval: Output Configuration Limitations

> Source: https://orval.dev/docs/reference/configuration/output/
> Collected: 2026-09-29
> Published: Unknown

## mode

Type: `'single' | 'split' | 'tags' | 'tags-split' | 'tags-operations' | 'tags-operations-split'` Default: `'single'`

### single

Everything in one file.

## urlEncodeParameters

Type: `Boolean` Default: `false`

Wrap each path parameter with `encodeURIComponent(String(...))` in generated URL helpers. This option only affects path parameters; query parameters are typically encoded by the underlying client (`URLSearchParams`, `axios`, etc.).

Path parameters are stringified via `String(value)` before encoding, so array (`style: simple|matrix|label`) and object path parameters are not serialized according to their OpenAPI `style` — they fall back to the default `String(value)` representation.
