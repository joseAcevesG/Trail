# Framework Review Groups

Use these groups to review the remaining design decisions one conversation at a time. Each group is sized to end with one new decision document and a checklist update.

## Completed Review Groups

### Group 1: HTTP Contract Surface

Completed. Consolidated decisions and checklist alignment are in:

- `http_contract_group1_wrapup.md`

## Suggested Next Group

### Group 2: Auth and API Security

This should be the next conversation.

Topics to decide:

- Authentication hooks
- Bearer token support
- API key support
- Session or cookie auth support
- Object-level authorization
- Field-level authorization
- Broken auth and broken authorization mitigations
- Rate limiting model
- Request timeout limits
- Concurrency limits
- `429 Too Many Requests`
- CORS policy
- Security headers
- HTTPS assumptions
- Sensitive data in URLs
- Header normalization
- Method allowlists
- Safe parsers
- SSRF protection
- Safe outbound HTTP helpers

Suggested file:

- `security_design.md`

## Remaining Review Groups

### Group 3: OpenAPI and API Evolution

Topics to decide:

- Spec-first support
- Auth schemes in OpenAPI
- Examples in OpenAPI
- Tags in OpenAPI
- Generated docs
- Contract checks
- SDK generation
- Versioning strategy
- Deprecation metadata
- Deprecation headers
- Breaking-change strategy

Suggested file:

- `openapi_and_versioning_design.md`

### Group 4: Collections and REST Query Features

Topics to decide:

- Pagination primitives
- Cursor pagination
- Offset pagination
- Stable ordering requirements
- `next` and `prev` pagination model
- Filtering conventions
- Sorting conventions
- Allowlisted filters
- Sparse fieldsets
- Query execution safety
- Cache-Control
- ETag
- Last-Modified
- `If-None-Match`
- `If-Match`

Suggested file:

- `collections_and_caching_design.md`

### Group 5: Runtime, Observability, and Deployment

Topics to decide:

- Structured logging
- Correlation IDs
- Metrics
- Tracing hooks
- OpenTelemetry alignment
- Health endpoint
- Readiness endpoint
- Startup and liveness semantics
- Dependency health checks
- Security event logs
- Log redaction
- Log injection protection
- Environment-based config
- Secret handling
- Config validation at startup
- Build, release, run separation
- Stateless process model
- Port binding
- Fast startup
- Backing services through config
- Startup and shutdown lifecycle hooks
- Request start and end lifecycle hooks

Suggested file:

- `runtime_and_operations_design.md`

### Group 6: Extensibility, Testing, and DX

Topics to decide:

- Dependency injection or service container
- Testing helpers
- Good defaults
- Plugin or module system
- Replaceable logger
- Replaceable validator
- Replaceable serializer
- Replaceable auth provider
- Framework metadata registry
- Unit test helpers
- Integration test helpers
- OpenAPI contract tests
- Security regression tests
- Error snapshot tests
- Middleware ordering tests
- Route reference docs
- Error catalog
- Auth guide
- Middleware or plugin guide
- Deployment guide
- Security recommendations

Suggested file:

- `extensibility_and_testing_design.md`

## Review Rule

Only check a box in the main checklist when the current design explicitly defines the behavior or clearly rejects an option.
