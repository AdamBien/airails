# HTTP Surface

Mapping of the [recording format](recording-format.md) to HTTP — REST endpoints and server-rendered pages.

- **Stimulus notation:** `GET /catalog/products?category=fish` — method, path, query. Request body for writes goes under a `## Request` section.
- **File naming:** `http_get_catalog_products_fish.md`.
- **Expected metadata:** `status` and `content-type` only, unless a flow depends on more — then list the kept headers explicitly in the recording.

## HTTP Normalization

On top of the generic masking rules:

- Drop volatile headers: `Set-Cookie`, `Date`, `ETag`, `Last-Modified`, `X-Request-Id`.
- HTML: collapse runs of whitespace between tags before comparing.
- JSON: pretty-print with sorted object keys before storing and comparing — member order is not part of the contract.
- Redirects: record the `Location` (normalized), do not follow automatically — the redirect is the behavior.
