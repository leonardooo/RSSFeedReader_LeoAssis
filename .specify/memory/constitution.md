<!--
SYNC IMPACT REPORT
==================
Version change: 1.0.0 → 1.1.0
Bump type: MINOR — material expansion of existing principles; new Principle VII added.

Modified principles:
  - Principle III: Security by Design — expanded with MUST/SHOULD rules specific to
    ASP.NET Core + Blazor; added HTTP security headers, URL allow-list approach,
    `dotnet list package --vulnerable` gate, no secrets in committed files.
  - Principle IV: Code Quality & Testability — added concrete tooling (xUnit, Moq,
    dotnet format, StyleCop.Analyzers); replaced vague targets with MUST language.
  - Principle V: Architecture for Scale — replaced passive voice with MUST/SHOULD.
  - Principle VI: Cross-Platform Compatibility — strengthened with MUST language.

Added sections:
  - Principle VII: Maintainability & Readability (new).
  - "Tooling Enforcement" subsection in Security & Quality Gates.

Removed sections:
  - None.

Templates requiring updates:
  ✅ plan-template.md — Constitution Check section references principles I–VI;
     Principle VII now exists. Template uses generic placeholder [Gates determined
     based on constitution file] — no hardcoded reference to remove; no edit needed.
  ✅ spec-template.md — No direct principle references; no edit needed.
  ✅ tasks-template.md — No direct principle references; no edit needed.

Follow-up TODOs:
  - None. All placeholders resolved.
-->

# RSS Feed Reader Constitution

## Core Principles

### I. MVP-First Incremental Development

Development MUST start with the smallest working slice—subscription list management
without feed fetching—and expand only after the current scope is verified end-to-end.
Each phase builds cleanly on the previous one:

1. **MVP**: Add and display subscriptions (in-memory, no feed fetching).
2. **Extended-MVP**: Manual feed fetch; display title + link per item.
3. **Production**: Persistence, background polling, advanced UX.

No Extended-MVP work may begin until MVP acceptance criteria are passing.

**Why**: Rapid feedback on architecture and UX; prevents over-engineering; enables
early detection of design flaws before large investment.

### II. Separation of Concerns (Backend API / Frontend UI)

The ASP.NET Core backend MUST own: subscription storage, feed operations, input
validation, and error propagation. The Blazor WebAssembly frontend MUST own: user
interaction, form state, and result display. Communication MUST occur exclusively
via the documented REST API contract. No business logic or data-access code is
permitted in Razor components.

**Why**: Enables independent scaling, testing, and deployment; maintains clean
boundaries; supports future multi-client scenarios (web, mobile, desktop).

### III. Security by Design

Security controls MUST be built in at every API boundary and MUST NOT be deferred
to post-MVP cleanup. Specific requirements:

- **Input validation**: Every API endpoint MUST reject malformed input with a 400
  response before any processing. Even in MVP, `POST /subscriptions` MUST verify
  the body is a non-empty string with a plausible URL format
  (`Uri.IsWellFormedUriString` with `UriKind.Absolute`).
- **CORS**: The backend MUST configure an explicit allow-list of origins.
  Wildcard `*` is PROHIBITED in all environments, including local development.
  Allowed origins are read from configuration, not hard-coded.
- **HTTP security headers**: All API responses MUST include at minimum:
  `X-Content-Type-Options: nosniff` and `X-Frame-Options: DENY`. Add via
  ASP.NET Core middleware, not per-controller.
- **No secrets in code or config files**: Connection strings, API keys, and tokens
  MUST be supplied through environment variables or a secrets manager. Committed
  `appsettings.*.json` files MUST NOT contain secret values.
- **Dependency scanning**: `dotnet list package --vulnerable` MUST be run before
  every pull-request merge. Critical or high-severity vulnerabilities MUST be
  resolved before merge; medium-severity within 48 hours of notification.

**Why**: Prevents OWASP Top 10 vulnerabilities (injection, misconfiguration,
vulnerable components); builds user trust; establishes secure defaults that remain
valid as the app grows toward production.

### IV. Code Quality & Testability

- **Test coverage**: All service-layer and domain logic MUST have xUnit unit tests.
  Coverage target is ≥ 75% on `backend/` measured by `dotnet test --collect:"XPlat
  Code Coverage"`. Use Moq for dependency isolation.
- **Test-first discipline**: Tests MUST be written before or alongside feature
  implementation. No PR merges a feature without tests for the acceptance criteria
  in the spec.
- **Code style**: `dotnet format` MUST produce zero changes on every PR. The project
  MUST include `StyleCop.Analyzers` and a `.editorconfig` enforcing consistent style.
- **CI enforcement**: Linting, formatting, and test gates run in CI (or pre-commit
  hooks). Builds MUST NOT be merged if any gate is red.
- **No dead code**: Unused imports, unreachable code paths, and commented-out blocks
  MUST be removed before merge.

**Why**: Reduces defects; enables confident refactoring; documents expected behavior
through executable tests; supports long-term maintenance as the codebase grows.

### V. Architecture for Scale

MVP code MUST be written so it does not require wholesale replacement for
Extended-MVP or production features:

- Entity models MUST be class-based (not `List<string>`) and MUST support EF Core
  attribute annotations without restructuring.
- The service layer (e.g., `ISubscriptionService`) MUST be abstracted from the
  storage mechanism via interfaces so the in-memory store can be swapped for EF
  Core without changing controllers.
- Background job concerns MUST be isolated behind a `IFeedPollingService`
  interface even if the MVP implementation is a no-op stub.

**Why**: Prevents costly rewrites; supports business growth; validates architectural
decisions early while the cost of change is low.

### VI. Cross-Platform Compatibility

All code MUST run unchanged on Windows, macOS, and Linux using only:
- .NET 8+ runtime
- ASP.NET Core and Blazor WebAssembly standard libraries
- NuGet packages with verified cross-platform support

Path construction MUST use `Path.Combine()` or forward-slash literals only.
Platform-specific APIs (`Registry`, `NSApplication`, etc.) are PROHIBITED.

**Why**: Maximises developer flexibility; supports CI on Linux runners; ensures
users on any OS can run the application.

### VII. Maintainability & Readability

Code MUST be written for the next developer, not only the current author:

- **Naming**: Classes, methods, and variables MUST use clear, domain-specific names
  (e.g., `SubscriptionService`, not `Service1`). Abbreviations are PROHIBITED
  except for universally understood terms (`url`, `id`, `dto`).
- **Method length**: Public methods MUST NOT exceed 30 lines. Extract private
  helpers when logic branches deeply or repeats.
- **Dependency injection**: All service dependencies MUST be injected via constructor
  (not `new`-ed inline or retrieved via service locator).
- **Configuration over magic strings**: Environment-specific values (ports, origins,
  timeouts) MUST be declared in `appsettings.json` / environment variables; no
  in-code string literals for configuration.
- **Changelog discipline**: Every PR description MUST summarise what changed and
  why; link to the relevant spec or task ID.

**Why**: Reduces onboarding friction; supports refactoring; makes diffs easier to
review; keeps the codebase approachable as features are added over time.

## Development Workflow

- **Branching**: Feature branches MUST follow the naming convention `###-feature-name`
  (e.g., `001-rss-subscriptions`).
- **Code Review**: All PRs require at least one review before merge. Constitution
  compliance is an explicit review checklist item.
- **Deployment**: MVP MUST be deployable as a self-contained binary or container.
  Extended-MVP and beyond MUST maintain this property.

## Security & Quality Gates

| Gate | Rule | Enforcement |
|---|---|---|
| Input validation | All API endpoints reject invalid input with HTTP 400 | PR review + integration test |
| CORS allow-list | No wildcard origin in any environment | Startup config audit |
| HTTP security headers | `X-Content-Type-Options`, `X-Frame-Options` on all responses | Integration test / middleware |
| No committed secrets | `appsettings.*.json` contains no credentials | Pre-commit hook / PR review |
| Dependency scan | Zero critical/high CVEs at merge time | `dotnet list package --vulnerable` in CI |
| Formatting | `dotnet format` diff is empty | CI gate |
| Test coverage | ≥ 75% on backend service layer | `dotnet test --collect:"XPlat Code Coverage"` |
| Dead code | No unused imports or commented-out blocks | StyleCop + PR review |

## Governance

This constitution supersedes all other development practices. Amendments require a
pull request documenting the change rationale and impact on existing artifacts. All
active team members MUST acknowledge amendments before they take effect.

**Complexity Exceptions**: Deviations from these principles require written
justification in the PR description (e.g., if a 4th service tier is needed beyond
Backend/Frontend/DataAccess, explain why the three-tier boundary is insufficient).
Exceptions are not blanket approvals; each exception is reviewed per-PR.

**Compliance Verification**: Constitution compliance is spot-checked during code
review and verified in full during each phase transition (MVP → Extended-MVP,
Extended-MVP → Production).

**Versioning Policy**:
- MAJOR: Backward-incompatible removal or redefinition of a principle.
- MINOR: New principle added or existing principle materially expanded.
- PATCH: Clarification, wording fix, or non-semantic refinement.

**Version**: 1.1.0 | **Ratified**: 2026-05-19 | **Last Amended**: 2026-05-20
