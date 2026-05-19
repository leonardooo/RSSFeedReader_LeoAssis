# RSS Feed Reader Constitution

## Core Principles

### I. MVP-First Incremental Development

Start with the smallest viable feature—subscription list management without feed fetching—and expand only when current scope is working and verified. Each phase builds cleanly on the previous: MVP (add/display subscriptions) → Extended-MVP (fetch and display feed items) → Production features (persistence, background polling, advanced UX).

**Why**: Rapid feedback on architecture and UX; prevents over-engineering; enables early detection of design flaws before large investment.

### II. Separation of Concerns (Backend API / Frontend UI)

Backend (ASP.NET Core) owns subscription storage, feed operations, and data validation. Frontend (Blazor WebAssembly) owns user interaction, form state, and result display. Communication strictly via REST API contract; no business logic in UI code.

**Why**: Enables independent scaling, testing, and deployment; maintains clean boundaries; supports future multi-client scenarios (web, mobile, desktop).

### III. Security by Design

Input validation is mandatory at API boundaries. CORS policy is explicit and minimal (localhost in dev, production origin in prod). Sensitive operations (feed refresh, data mutations) are guarded by validation. No secrets in code or config files.

**Why**: Prevents common web vulnerabilities; builds user trust; supports compliance requirements for future production versions.

### IV. Code Quality & Testability

All new components must have unit test coverage for core logic (>75% target). Integration tests validate API contracts and user journeys. Tests are written before or alongside implementation. Linting and formatting rules are enforced in CI/pre-commit hooks.

**Why**: Reduces defects; enables confident refactoring; documents expected behavior; supports long-term maintenance.

### V. Architecture for Scale

Design with future production features in mind: entity models support database persistence, service layer is abstracted from storage, background jobs are isolated in a separate concern. Even MVP code should not require wholesale replacement for Extended-MVP features.

**Why**: Prevents costly rewrites; supports business growth; validates architectural decisions early.

### VI. Cross-Platform Compatibility

Code runs unchanged on Windows, macOS, and Linux. Use only cross-platform dependencies (ASP.NET Core, Blazor, .NET runtime). Avoid platform-specific file paths or APIs; use abstractions instead.

**Why**: Maximizes user reach; supports development across teams; maintains developer flexibility in tool choice.

## Development Workflow

- **Branching**: Feature branches use naming convention `###-feature-name` (e.g., `001-subscription-add`).
- **Code Review**: All PRs require review before merge; constitution compliance is a review criterion.
- **Deployment**: MVP is deployable (self-contained single binary or containerized); Extended-MVP and beyond maintain this property.

## Security & Quality Gates

- **Input Validation**: All API endpoints validate input schema and type before processing.
- **CORS Policy**: Explicitly configured per environment; no wildcard `*` in production.
- **Logging**: All errors and security-relevant events logged with sufficient context for troubleshooting.
- **Dependency Management**: Vulnerable dependencies are patched within 48 hours of notification.

## Governance

This constitution supersedes all other development practices. Amendments require a pull request documenting the change rationale and impact on existing artifacts. All active team members must acknowledge amendments before they take effect.

**Complexity Exceptions**: Deviations from these principles require written justification in the PR description (e.g., if a 4th tier of service classes is needed beyond BackendAPI/Frontend/DataAccess, explain why the three-tier boundary is insufficient). Exceptions are not blanket approvals; each exception is reviewed per-PR.

**Compliance Verification**: Constitution compliance is spot-checked during code review and verified in full during each Extended-MVP phase transition.

**Version**: 1.0.0 | **Ratified**: 2026-05-19 | **Last Amended**: 2026-05-19
