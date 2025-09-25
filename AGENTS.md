
# Development Guidelines

General development guidelines for this project:

## Philosophy

### Core Beliefs

- **Incremental progress over big bangs** - Small changes that compile and pass tests
- **Learning from existing code** - Study and plan before implementing
- **Pragmatic over dogmatic** - Adapt to project reality, real-world focus
- **Clear intent over clever code** - Be boring and obvious

### Simplicity Means

- Single responsibility per function/class
- Avoid premature abstractions
- No clever tricks - choose the boring solution
- Avoid unnecessary complexity, if you need to explain it, it's too complex
- Avoid unreadable code

## Process

### 1. Planning & Staging

Break complex work into 3-5 stages. And document key designs in `designs/IMPLEMENTATION_design.md`:

```markdown
## Stage N: [Name]
**Goal**: [Specific deliverable]
**Success Criteria**: [Testable outcomes]
**Tests**: [Specific test cases]
**Status**: [Not Started|In Progress|Complete]
```
- Update status as you progress
- Remove file when all stages are done

### 2. Implementation Flow

1. **Understand** - Study existing patterns in codebase
2. **Test** - Write test first (red)
3. **Implement** - Minimal code to pass (green)
4. **Refactor** - Clean up with tests passing
5. **Commit** - With clear message linking to plan

### 3. When Stuck (After 3 Attempts)

**CRITICAL**: Maximum 3 attempts per issue, then STOP.

1. **Document what failed**:
   - What you tried
   - Specific error messages
   - Why you think it failed

2. **Research alternatives**:
   - Find 2-3 similar implementations
   - Note different approaches used

3. **Question fundamentals**:
   - Is this the right abstraction level?
   - Can this be split into smaller problems?
   - Is there a simpler approach entirely?

4. **Try different angle**:
   - Different library/framework feature?
   - Different architectural pattern?
   - Remove abstraction instead of adding?

## Technical Standards

### Architecture Principles

- **Composition over inheritance** - Use dependency injection
- **Interfaces over singletons** - Enable testing and flexibility
- **Explicit over implicit** - Clear data flow and dependencies
- **Test-driven when possible** - Never disable tests, fix them

### Code Quality

- **Every commit must**:
  - Compile successfully
  - Pass all existing tests
  - Include tests for new functionality
  - Follow project formatting/linting
  - No performance regressions

- **Before committing**:
  - Run formatters/linters
  - Self-review changes
  - Ensure commit message explains "why"

### Error Handling

- Fail fast with descriptive messages
- Include context for debugging
- Handle errors at appropriate level
- Never silently swallow exceptions

## Decision Framework

When multiple valid approaches exist, choose based on:

1. **Testability** - Can I easily test this?
2. **Readability** - Will someone understand this in 6 months?
3. **Consistency** - Does this match project patterns?
4. **Simplicity** - Is this the simplest solution that works?
5. **Reversibility** - How hard to change later?

## Project Integration

### Learning the Codebase

- Find 3 similar features/components
- Identify common patterns and conventions
- Use same libraries/utilities when possible
- Follow existing test patterns

### Tooling

- Use project's existing build system
- Use project's test framework
- Use project's formatter/linter settings
- Don't introduce new tools without strong justification

## Quality Gates

### Definition of Done

- Tests written and passing
- Code follows project conventions
- No linter/formatter warnings
- Commit messages are clear
- Implementation matches plan
- No TODOs without issue numbers

### Test Guidelines

- Test behavior, not implementation
- One assertion per test when possible
- Clear test names describing scenario
- Use existing test utilities/helpers
- Tests should be deterministic

## Important Reminders

**NEVER**:
- Use `--no-verify` to bypass commit hooks
- Disable tests instead of fixing them
- Commit code that doesn't compile
- Make assumptions - verify with existing code

**ALWAYS**:
- Commit working code incrementally
- Update plan documentation as you go
- Learn from existing implementations
- Stop after 3 failed attempts and reassess


# Repository Guidelines(for Agents and Maintainers)

This repository hosts Helm charts for Agora services, notably `charts/rtc-egress`.
Use this guide when proposing changes, diagnosing issues, or updating docs.

## Project Overview
- Chart: `charts/rtc-egress` (separated architecture recommended; monolithic legacy).
- Published Helm repo: `https://helm.agora.build`.
- Images are built in the RTC‑Egress source repo and pushed to GHCR:
  `ghcr.io/agoraio/rtc-egress/<service>:<tag>`.
- Chart versions live in `charts/rtc-egress/Chart.yaml` (`version`, `appVersion`).

## Configuration Design (must stay consistent)
- Each service reads a YAML file mounted at `/opt/rtc_egress/config` and selected via
  `CONFIG_FILE` env:
  - `api_server_config.yaml`
  - `egress_config.yaml`
  - `flexible_recorder_config.yaml`
  - `uploader_config.yaml`
  - `webhook_notifier_config.yaml`
- Resolution order (already implemented in binaries):
  `CONFIG_FILE` > `--config` > `CONFIG_DIR/<file>` > search `./config`, `/opt/rtc_egress/config`, `/etc/rtc_egress`.
  Services log the chosen path and fail fast if none found.
- Health ports (defaults, value‑driven): API 8191; Egress 8192; Flexible 8193; Uploader 8194; Notifier 8195.
- Redis worker patterns are rendered from values into config; defaults include global + regional patterns.
- S3: Helm-only toggle `s3.enabled`; code reads s3.bucket/region/keys/endpoint (no `enabled` key in YAML required).

## Mandatory Values (for users of this chart)
See the "Mandatory Parameters" in `charts/rtc-egress/README.md`.

## When Changing the Chart
- Keep changes minimal and backwards‑compatible where practical.
- If you change templates, values, or rendered config, bump `charts/rtc-egress/Chart.yaml:version`.
  If you release new binaries, bump `appVersion` accordingly.
- Update docs alongside changes:
  - `charts/rtc-egress/README.md` (installation, mandatory params, config resolution, health ports).
  - `charts/rtc-egress/AUTOSCALING.md` (HPA + queue patterns assumptions).
  - Root `README.md` and `REPOSITORY-SETUP.md` if relevant.
- Validate locally:
  - `helm lint charts/rtc-egress`
  - `helm template charts/rtc-egress > out.yaml` and inspect ConfigMap keys and env `CONFIG_FILE`.
- Do not commit secrets. Use Kubernetes Secrets or CI‑injected values.

## Common Gotchas
- Probe/port mismatches: ensure containerPort + liveness/readiness match values.
- Config schema drift: keep ConfigMap keys in sync with the binaries (e.g., `server.health_port`, `watcher.directories`).
- HPA queueName vs worker_patterns: ensure you monitor the same namespace you subscribe to.
- Using `image.tag`: per‑service tags take precedence; otherwise the chart falls back to `image.tag`.

## Release & Repo Publishing
- Pushing to `main` triggers chart packaging and publishing to GitHub Pages (`helm.agora.build`).
- Verify after release:
  - `helm repo update`
  - `helm search repo agora/rtc-egress --versions | head`

## How Agents Should Operate Here
- Prefer surgical changes; do not alter build workflows unless explicitly requested.
- When fixing runtime issues:
  - Check the rendered ConfigMap and `CONFIG_FILE` env per service.
  - Confirm health ports and probes.
  - Mention mandatory values a user must set (Agora, Redis, webhook URL, S3).
- Always propose doc updates for user‑visible changes.
