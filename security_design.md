# Security Design (Group 2)

## Scope

This document records decisions for Group 2 in small parts.

## Part 2.1: Authentication Modes and Middleware Model

### Decisions

- Trail supports multiple authentication modes:
  - Bearer token
  - API key
  - Session/cookie
- Each mode is optional and can be enabled independently.
- Trail uses a two-layer authentication flow:
  1. Global auth-resolution middleware parses credentials, validates configured schemes, and attaches user/auth context when successful.
  2. Route-level `requireAuth` middleware enforces that authenticated user context is present.
- On routes that do not require authentication, invalid provided credentials still fail the request (`401 Unauthorized`) instead of silently downgrading to anonymous.

## Part 2.2: Auth Provider Model

### Decisions

- Trail supports two auth provider modes:
  - `better-auth`: Trail provides a configurable built-in structure powered by Better Auth.
  - `custom`: application defines its own auth structure against Trail's provider contract.
- Trail defines a stable auth provider contract so framework middleware behavior remains consistent across providers.
- Better Auth integration is a default implementation, not a hard lock-in.
- Trail should provide an auth setup generator command (`npx trail auth`) to scaffold provider selection, auth mode choices, and middleware wiring.

### Provider Contract (high level)

- Resolve identity from request.
- Validate configured credential types.
- Expose auth context to middleware and handlers.
- Integrate with authorization policy checks.

### Rationale

- Teams can move fast with a secure default while keeping freedom to adopt custom auth models.
- A stable contract prevents framework-level auth drift and keeps DX predictable.


## Part 2.3: Rate Limiting and Request Protection

### Decisions

- Trail ships opinionated defaults, while allowing full developer overrides.
- Default rate limit dimensions: per-IP, per-token, and per-route.
- Default algorithm: token bucket.
- Default `429 Too Many Requests` behavior includes `Retry-After` and Trail standard error body/code.
- Default execution controls:
  - Global request timeout with per-route override.
  - Global concurrency cap with per-route override.
- If limiter backend is unavailable, default behavior is hybrid:
  - Fail closed for protected/high-risk routes.
  - Fail open for public routes.

- Rate limiting storage is backend-agnostic through a store adapter contract (no Redis lock-in).
- Trail should provide first-party store adapters for local memory and Redis, and support custom adapters for other backends.
- In multi-instance deployments without a distributed store, Trail should emit a startup warning.

### Rationale

- Strong defaults reduce insecure deployments.
- Override support preserves flexibility for product-specific traffic models.
- Hybrid outage mode balances security and availability during incidents.


## Part 2.4: CORS, Security Headers, and HTTPS Posture

### Decisions

- Trail uses environment-aware defaults: lax for development speed, strict for production safety.
- Production defaults:
  - CORS deny-by-default with explicit origin allowlist.
  - No wildcard origins when credentials are enabled.
  - Security headers preset enabled by default.
  - HTTPS required in production, with trusted proxy configuration support.
  - Guardrails to discourage sensitive data in URLs.
- Development defaults:
  - Permissive localhost-focused CORS preset for rapid startup.
  - HTTPS not required.
  - Relaxed/minimal security headers.
  - Clear startup warning that development profile is not production-safe.
- All defaults are overrideable by explicit developer configuration.

### Rationale

- Reduces startup friction for local development.
- Maintains secure-by-default behavior where risk is highest (production).

## Detailed Explanations and Trade-offs

### Authentication Modes

- **Bearer token**
  - **What it is for:** stateless API authentication, common for mobile/frontend-to-API and service-to-service requests.
  - **How it works:** client sends `Authorization: Bearer <token>`, framework validates signature/session mapping, then attaches identity context.
  - **Trade-offs:** simple at runtime and scalable, but token revocation/rotation strategy must be designed.

- **API key**
  - **What it is for:** server-to-server integrations, automation clients, and low-friction machine auth.
  - **How it works:** client sends key in header (preferred), framework validates key metadata and maps to principal/scopes.
  - **Trade-offs:** easy onboarding for integrators, but weaker user-level attribution unless paired with scopes, rotation, and usage tracking.

- **Session/cookie**
  - **What it is for:** browser-based apps with login sessions and first-party web flows.
  - **How it works:** secure cookie identifies a server-side or signed session; middleware resolves user from session.
  - **Trade-offs:** good browser UX and revocation control, but requires CSRF/session-hardening patterns.

### Two-layer Auth Middleware

- **Global auth-resolution middleware**
  - **For:** parse credentials once and normalize identity context for all routes.
  - **How:** try configured schemes, validate presented credentials, attach context.
  - **Trade-off:** central consistency, but requires careful ordering with body/parser/error middleware.

- **Route-level `requireAuth` middleware**
  - **For:** explicit protection per route/group.
  - **How:** checks auth context presence/validity and rejects unauthorized access.
  - **Trade-off:** clear intent and flexibility, but teams must apply it consistently on protected endpoints.

### Provider Model (`better-auth` and `custom`)

- **`better-auth` mode**
  - **For:** secure fast-start defaults with less custom plumbing.
  - **How:** Trail ships configurable integration and wiring templates.
  - **Trade-off:** fast delivery, with dependency on provider ecosystem features.

- **`custom` mode**
  - **For:** domain-specific auth models and enterprise constraints.
  - **How:** app implements Trail auth-provider contract.
  - **Trade-off:** maximum flexibility, but more implementation and maintenance effort.

### Rate Limiting

- **Per-IP limits**
  - **For:** anonymous abuse protection and bot throttling.
  - **How:** bucket counters keyed by source IP.
  - **Trade-off:** can penalize shared NAT/proxy users.

- **Per-token limits**
  - **For:** fairness and abuse control per authenticated principal.
  - **How:** bucket counters keyed by token principal/subject.
  - **Trade-off:** requires robust identity extraction and token lifecycle handling.

- **Per-route limits**
  - **For:** protect expensive or sensitive endpoints.
  - **How:** route-specific policies override global thresholds.
  - **Trade-off:** more knobs to tune and document.

- **Token bucket algorithm**
  - **For:** balanced burst tolerance with steady-state control.
  - **How:** tokens refill over time; request consumes token if available.
  - **Trade-off:** excellent operational behavior, slightly more complex than fixed-window.

- **`429` with `Retry-After`**
  - **For:** predictable client backoff and reduced retry storms.
  - **How:** include retry timing and standard error code/body.
  - **Trade-off:** more implementation detail, better client interoperability.

- **Timeout + concurrency controls**
  - **For:** bounded resource consumption and latency protection.
  - **How:** global defaults with per-route overrides.
  - **Trade-off:** safer operations, but requires route classification/tuning.

- **Backend-agnostic rate-limit store**
  - **For:** avoid Redis lock-in and support diverse infrastructures.
  - **How:** adapter interface with memory/redis/custom implementations.
  - **Trade-off:** portability and flexibility, with added adapter contract complexity.

### CORS, Headers, and HTTPS

- **Explicit CORS allowlist in production**
  - **For:** prevent unintended cross-origin access.
  - **How:** deny by default unless origin is explicitly allowed.
  - **Trade-off:** safer posture, more initial config.

- **No wildcard with credentials**
  - **For:** prevent unsafe browser credential sharing across origins.
  - **How:** enforce exact origin matching when cookies/auth headers are allowed.
  - **Trade-off:** stricter integration setup, better security guarantees.

- **Security headers preset**
  - **For:** baseline browser hardening.
  - **How:** apply recommended headers by default, allow overrides.
  - **Trade-off:** safer defaults, occasional compatibility tuning.

- **HTTPS required in production**
  - **For:** protect credentials/data in transit.
  - **How:** enforce secure transport assumptions; support trusted proxy deployments.
  - **Trade-off:** operational TLS setup required, critical security benefit.

- **Dev-lax / Prod-strict profiles**
  - **For:** rapid local start without sacrificing production security.
  - **How:** environment profile switches defaults while preserving explicit overrides.
  - **Trade-off:** great DX, but requires clear warnings to avoid misconfigured deployments.


## Part 2.5: SSRF Protection and Safe Outbound HTTP

### Decisions

- Trail exposes outbound security controls as fully configurable toggles; developers can enable/disable each control explicitly.
- To avoid hidden latency costs, advanced outbound checks that add measurable overhead are **off by default**.
- Trail provides an easy opt-in path (config + CLI scaffolding) to enable recommended SSRF protections.
- Baseline model includes optional controls for:
  - Safe outbound helper usage (`safeFetch`).
  - Blocking loopback/private/link-local targets.
  - DNS resolve + connect-time revalidation.
  - Redirect hop limits and per-hop revalidation.
  - Metadata endpoint blocking.
  - Allowlist-based outbound policies.
- When protections are disabled, Trail should emit clear startup/runtime warnings so the trade-off is visible.

### Rationale

- Keeps latency and behavior explicit for developers.
- Preserves advanced security hardening for teams that need it.
- Aligns with Trail's flexibility principle: secure capabilities available, never hidden lock-in or forced overhead.

## Part 2.6: Better Auth Aligned Provider Proposal (Opt-in)

### Intent

This proposal applies only when a developer chooses Trail's built-in auth path.
If the developer does not choose that path, Trail provides only interfaces/contracts to implement.

### Mode A: `better-auth` (Opt-in implementation path)

When selected, Trail should provide a first-party integration aligned with common industry standards:

- OAuth 2.1 / OpenID Connect compatible login and token flows where applicable.
- Session hardening defaults for browser flows (secure cookies, same-site policy, rotation support).
- Token lifecycle support (expiry, refresh, rotation/revocation hooks).
- Credential transport best practices (auth in headers/cookies, no secrets in URLs).
- Standardized auth error mapping (`401` unauthenticated, `403` unauthorized).
- Audit-friendly auth events (login success/failure, logout, token refresh/revocation).

#### What Trail should generate in this mode

- `npx trail auth` scaffolds Better Auth wiring, provider config, middleware registration, and starter policy hooks.
- Environment-aware defaults (dev-friendly, production-safe).
- Clear extension points so teams can override defaults without forking framework internals.

### Mode B: `custom` (Interface-only path)

If Better Auth is not selected, Trail does not impose a concrete auth implementation.
Trail only requires that the app implements framework interfaces.

#### Required interfaces/contracts

1. **Identity Resolver**
   - Input: request context
   - Output: authenticated principal or anonymous context
   - Responsibility: parse/validate credentials and attach normalized auth context

2. **Auth Guard**
   - Responsibility: enforce authenticated access for protected routes (`requireAuth` semantics)

3. **Authorization Policy Hook**
   - Responsibility: evaluate action/resource access (`allow`/`deny`) for object/function/field checks as configured

4. **Credential Strategy Adapters**
   - Responsibility: support one or more schemes (bearer, api key, session) under a consistent interface

5. **Security Event Emitter**
   - Responsibility: emit structured auth/audit events to app logger/telemetry

6. **Error Mapper**
   - Responsibility: map auth failures to Trail's standard error shape and HTTP semantics

### Compatibility and portability goals

- No lock-in to Better Auth for application domain logic.
- Consistent middleware and authorization behavior regardless of selected provider mode.
- Swap path: teams can migrate from `better-auth` to `custom` (or back) by re-implementing interfaces, not rewriting route handlers.

### Trade-offs

- `better-auth` mode: faster onboarding and safer defaults, with dependency on provider ecosystem behavior.
- `custom` mode: maximum flexibility and control, with higher implementation and maintenance cost.

### Acceptance criteria

- Choosing `better-auth` yields a runnable secure baseline with minimal setup.
- Choosing `custom` requires only interface compliance; no hidden runtime coupling to Better Auth.
- Documentation clearly separates "implementation path" from "contract path".
