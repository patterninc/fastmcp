# Engineering Best Practices Audit — patterninc/fastmcp

| | |
|---|---|
| **Audit date** | 2026-09-08 |
| **Auditor** | Claude — gauge-repo skill |
| **Rubric version** | `item-credit-v1` — 2026-09-04 (`references/best-practices.md`) |

## Repo profile

`patterninc/fastmcp` is a verified GitHub fork of `PrefectHQ/fastmcp`, the standard Python framework (Python >= 3.10) for building Model Context Protocol servers and clients. It is a **library/SDK published as a package** (PyPI publishing is driven upstream via trusted publishing); Pattern's fork carries a single local commit ("Onboard fastmcp to Backstage", adding `backstage.yaml` owned by `ai-infra-comms`, system `aiplatform`) on top of upstream history, so nearly all repo-local practices are inherited upstream files that live in this repo and therefore count as evidence. There is **no deployment, no database, no browser UI, and no AWS footprint** in this repo — nothing here is operated as a service by Pattern. Tooling is uv-centric (`uv.lock`, `.python-version`, `justfile`), CI is GitHub Actions (`run-tests.yml`, `run-static.yml`, `run-integration-tests` job, `publish.yml`), and quality gates run through prek/pre-commit (ruff, ruff-format, prettier for YAML/JSON5, ty, loq, codespell). Ownership was verified with the GitHub API (`patterninc/fastmcp`, `isFork: true`, parent `PrefectHQ/fastmcp`), so Pattern inherited controls apply: org-wide Wiz (items 19, 20, 47) and Toolsmith-managed MCP access (item 39). An active org-level branch ruleset (`require-pr-review`, id 3174764) applies to the default branch. Recommendations below are kept proportionate to the fork's tracking role — repo-local additions that would fight upstream syncs are flagged as such.

No previous audit exists at this path; this is the first audit.

## Scorecard

| Metric | Value |
|--------|-------|
| **Critical gates** | **RED** |
| **Adjusted compliance** | **71.3%** |

Critical gates are RED because item 16 (required CI checks before merge) is **Partial**: CI workflows run on every PR, but the active ruleset contains no required-status-checks rule and the classic protection view shows no required checks, so merges are not verifiably blocked on green CI. All other applicable gates (2, 6, 15, 19, 20, 23, 24, 48) are Met; gate 40 is a justified N/A.

`(24 Met + 0.5 × 9 Partial) / (49 total - 9 justified N/A) = 28.5 / 40 = 71.3%`

### Status totals

| Status | Items |
|--------|------:|
| Met | 24 |
| Partial | 9 |
| Gap | 7 |
| N/A | 9 |
| **Total** | **49** |

### Per-category breakdown

| Category | Met | Partial | Gap | N/A |
|----------|----:|--------:|----:|----:|
| Documentation & Context | 5 | 2 | 1 | 1 |
| Guardrails & Enforcement | 7 | 3 | 2 | 1 |
| Testing & Feedback Loops | 5 | 3 | 2 | 3 |
| Environment & Tooling | 7 | 1 | 1 | 4 |
| Agent dispatch | 0 | 0 | 1 | 0 |
| **Total** | **24** | **9** | **7** | **9** |

## Documentation & Context

| # | Practice | Status | Evidence | Recommendation / rationale |
|---|----------|--------|----------|----------------------------|
| 1 | Skills / reusable prompt workflows | **Met** | `.claude/skills/code-review`, `.claude/skills/python-tests`, `skills/fastmcp-client-cli/SKILL.md`, `.claude/hooks/session-init.sh` | — |
| 2 | AGENTS.md | **Met** | `AGENTS.md` (symlink to `CLAUDE.md`) with required workflow, repo map, dev rules, critical patterns; plus `.github/copilot-instructions.md` and `.cursor/rules/` | — |
| 3 | Architecture decision records | **Partial** | `v3-notes/` design notes (`provider-architecture.md`, `visibility.md`, etc.) capture v3 design rationale | Design notes exist but are not dated, immutable ADRs. Adopt a lightweight `docs/adr/` convention for future structural decisions — especially any Pattern-local divergence from upstream. |
| 4 | Runbooks | **Partial** | `docs/development/releases.mdx` documents the release process; `docs/development/contributing.mdx`, `docs/development/tests.mdx` | Add a Pattern-specific fork-maintenance runbook: how and when to sync from `PrefectHQ/fastmcp`, how to carry local commits, and how conflicts are resolved. |
| 5 | API contract docs | **Met** | Generated JSON Schemas in `docs/public/schemas/` and `src/fastmcp/utilities/mcp_server_config/v1/schema.json` (bot-maintained per `CLAUDE.md`); the wire contract is the external MCP specification implemented via the `mcp` SDK | — |
| 6 | README with setup & run instructions | **Met** | `README.md` (install, quickstart, docs, upgrade guides); dev setup in `CLAUDE.md` (`uv sync`, `uv run pytest -n auto`) and `justfile` | — |
| 7 | Changelog with migration notes | **Met** | `docs/changelog.mdx` (per-release entries), upgrade guides linked from `README.md` (from v2, from MCP SDK) | — |
| 8 | On-call playbooks | **Not applicable** | Library fork; nothing operated in production from this repo | No service, no incidents, no pager. Release docs (item 4) cover the operational surface that exists. |
| 9 | CODEOWNERS | **Gap** | No `CODEOWNERS` file; `backstage.yaml` names owner `ai-infra-comms` but GitHub does not auto-assign from it | Add `.github/CODEOWNERS` mapping `*` to the `ai-infra-comms` team so the org ruleset's required review routes to the owning team. |

## Guardrails & Enforcement

| # | Practice | Status | Evidence | Recommendation / rationale |
|---|----------|--------|----------|----------------------------|
| 10 | Linters | **Met** | ruff with bugbear/comprehensions/pie/sim/ruf extensions (`pyproject.toml`), enforced in `.pre-commit-config.yaml` and `.github/workflows/run-static.yml`; codespell | — |
| 11 | Formatters | **Met** | `ruff-format` plus prettier for YAML/JSON5 in `.pre-commit-config.yaml`, enforced in CI via prek | — |
| 12 | Type checking | **Met** | ty with `error-on-warning = true` over `src` and `tests` (`pyproject.toml`), in pre-commit and `run-static.yml`; package ships `py.typed` | — |
| 13 | Pre-commit hooks | **Met** | `.pre-commit-config.yaml` run via prek; install and usage documented as required workflow in `CLAUDE.md` | — |
| 14 | Commit message conventions | **Partial** | `CLAUDE.md` documents commit-message and PR-label conventions; `.github/release.yml` drives label-based release notes | Guidance exists but nothing enforces format or labels. If Pattern wants automation from history, add a commitlint or PR-title check; otherwise document that labels are the enforced convention. |
| 15 | Branch protection rules | **Met** | Active org ruleset `require-pr-review` (id 3174764) on the default branch: PR required, 1 approval, stale-review dismissal, no deletion, no force push; local `no-commit-to-branch` hook as backup | — |
| 16 | Required CI checks before merge | **Partial** | `run-tests.yml` and `run-static.yml` trigger on all PRs (comments say they are intended as required checks), but the ruleset contains no required-status-checks rule and the classic protection view reports no required checks | Add required status checks (Tests matrix + static analysis) to a repo-level ruleset on `main` so merges are blocked until CI is green. This is the sole RED critical gate. |
| 17 | Dependency allow/deny lists | **Not applicable** | Fork tracks upstream's dependency set (`pyproject.toml` is upstream-owned) | A Pattern-local allow/deny policy would block every upstream sync for zero benefit; dependency policy for Pattern services applies where fastmcp is consumed, not in the tracking fork. |
| 18 | License compliance scanning | **Gap** | No license-check job; Apache-2.0 project with a sizable dependency tree | Add a periodic license scan (e.g., `pip-licenses` or Wiz license policy if available) — low priority, and prefer an org-side control to avoid CI drift from upstream. |
| 19 | Secret scanning | **Met** | Inherited Pattern Wiz policy (org-wide secret scanning; ownership verified via GitHub API) | — |
| 20 | SAST / static analysis gates | **Met** | Inherited Pattern Wiz policy (org-wide SAST with blocking enforcement) | — |
| 21 | Max complexity limits | **Partial** | loq file-size limits with per-file ratchet (`loq.toml`), wired into pre-commit — but the hook is advisory ("violations not enforced... yet!") and no cyclomatic-complexity rule (ruff C901/PLR) is enabled | Make the loq hook fail on violations, or enable ruff `C901` with a generous ceiling. |
| 22 | Import boundary enforcement | **Gap** | `CLAUDE.md` documents module re-export discipline; ruff isort configured; no import-linter or banned-API rules | Encode the documented export/import rules (e.g., ruff `TID` banned-imports or import-linter contracts for server vs client). Weigh against fork-sync drift; consider proposing upstream instead. |

## Testing & Feedback Loops

| # | Practice | Status | Evidence | Recommendation / rationale |
|---|----------|--------|----------|----------------------------|
| 23 | Unit tests | **Met** | 332 test files under `tests/`; CI matrix on ubuntu/windows and Python 3.10/3.13 plus a lowest-direct-dependencies job (`run-tests.yml`) | — |
| 24 | Integration tests | **Met** | `tests/integration_tests/`, `integration` pytest marker, dedicated `run_integration_tests` CI job via `.github/actions/run-pytest` | — |
| 25 | Snapshot / golden-file tests | **Met** | `inline-snapshot` dev dependency; snapshot assertions across `tests/` (e.g., `tests/utilities/openapi/test_models.py`); CI runs with `--inline-snapshot=disable` so snapshots act as fixed assertions | — |
| 26 | Contract tests | **Met** | Equivalent: client and server are exercised against each other over in-memory MCP transport throughout `tests/`, verifying both sides of the MCP wire contract; `tests/test_json_schema_generation.py` and `tests/test_mcp_config.py` validate generated schemas | — |
| 27 | End-to-end tests (Playwright) | **Not applicable** | No owned browser UI — this is a protocol library | Client-server round-trip and integration tests cover the equivalent full-flow surface. |
| 28 | Visual regression tests | **Not applicable** | No owned visual surface; app UIs render in MCP host clients, not in this repo | — |
| 29 | Test coverage thresholds | **Gap** | `pytest-cov` is a dev dependency but no coverage run or threshold exists in CI (`.github/actions/run-pytest/action.yml`) | Add a coverage report with a ratcheted minimum to the ubuntu CI leg. Prefer proposing upstream to avoid workflow drift. |
| 30 | Mutation testing | **Gap** | None found | Low priority: a scoped `mutmut` run over core modules (`src/fastmcp/server/`, `src/fastmcp/tools/`) on a schedule would validate test strength for a widely-depended-on library. |
| 31 | Load / performance benchmarks | **Partial** | `scripts/benchmark_imports.py` guards import-time cost | No throughput/latency benchmarks or CI regression tracking for server/client hot paths. Add a small pytest-benchmark suite for request dispatch if performance regressions become a concern. |
| 32 | Flaky test quarantine | **Partial** | `pytest-retry` and `pytest-flakefinder` dev dependencies; `client_process` marker isolates subprocess-flaky tests into a separate CI step; strict 5s default timeout | Tooling exists but there is no documented quarantine mechanism (e.g., a `flaky` marker excluded from required runs). Document the policy. |
| 33 | Structured CI output | **Partial** | Structured job matrix with per-job pass/fail; `pytest-report` dev dependency | Test runs emit plain text only — no JUnit XML artifacts or GitHub annotations. Add `--junitxml` upload to `run-pytest` for machine-readable failures. |
| 34 | Deterministic test fixtures | **Met** | `pytest-env` pins `FASTMCP_TEST_MODE=1` and log settings (`pyproject.toml`), function-scoped asyncio loops, in-memory transports, shared `tests/conftest.py` fixtures | — |
| 35 | Smoke tests for deploys | **Not applicable** | Library with no deployment from this repo; publishing happens upstream on release | Post-install verification is covered by the CI lowest-direct-dependencies job and upstream release process. |

## Environment & Tooling

| # | Practice | Status | Evidence | Recommendation / rationale |
|---|----------|--------|----------|----------------------------|
| 36 | Devcontainer config | **Gap** | No `.devcontainer/`; environment is well-pinned via uv, `.python-version`, and `uv.lock` | Optional: add a devcontainer wrapping `uv sync` for uniform agent/human onboarding. Low value given uv already yields near-identical environments. |
| 37 | One-command setup | **Met** | `uv sync` bootstraps everything; `justfile` targets (`build`, `test`, `typecheck`, `docs`); documented as required workflow in `CLAUDE.md` | — |
| 38 | Seed scripts for local databases | **Not applicable** | No database anywhere in the repo | — |
| 39 | MCP servers for external tools | **Met** | Inherited: Toolsmith-managed MCP access for Pattern repos; additionally the repo is itself an MCP framework with runnable servers under `examples/` | — |
| 40 | Scoped secrets per environment | **Not applicable** | No dev/staging/prod environments or owned credentials; the only credentialed workflow (`publish.yml`) uses PyPI trusted publishing via OIDC `id-token`, storing no long-lived secret | — |
| 41 | Preview environments per PR | **Not applicable** | Library; nothing to deploy per PR | CI matrix serves the per-PR verification role. |
| 42 | Hot-reload / watch mode | **Met** | `fastmcp run --reload` and `--reload-dir` (`src/fastmcp/cli/cli.py`) backed by `watchfiles`; `just docs` runs Mintlify dev server | — |
| 43 | Structured logging (JSON) | **Partial** | `src/fastmcp/utilities/logging.py` provides configurable logging (RichHandler, human-readable); server logging middleware exists | No JSON formatter option. As a library it correctly defers to the host app, but offering a built-in JSON handler option would help consumers ship queryable logs. Consider proposing upstream. |
| 44 | Observable traces and metrics | **Met** | OpenTelemetry integration: `opentelemetry-api` core dependency, `src/fastmcp/telemetry.py`, `src/fastmcp/server/telemetry.py`, `src/fastmcp/client/telemetry.py`, `tests/telemetry/` | — |
| 45 | Feature flags with local overrides | **Met** | Equivalent: pydantic-settings runtime toggles with `FASTMCP_` env prefix and `.env` file overrides (`src/fastmcp/settings.py`); `experimental` namespace gates in-progress features | — |
| 46 | Database migration tooling | **Not applicable** | No database | — |
| 47 | Dependency update automation | **Met** | Inherited org-wide Wiz (verified Pattern repo); `.github/dependabot.yml` (pip daily, actions weekly) also present from upstream | — |
| 48 | Reproducible builds (lockfiles) | **Met** | `uv.lock` committed; CI installs with `resolution: locked` (`.github/actions/setup-uv`); `justfile` uses `--frozen`; `.python-version` pins the interpreter | — |

## Agent dispatch

| # | Practice | Status | Evidence | Recommendation / rationale |
|---|----------|--------|----------|----------------------------|
| 49 | Agent-dispatch manifest | **Gap** | No `.agents/pattern-agents.json`; `backstage.yaml` exists but is not the dispatch manifest | Add `.agents/pattern-agents.json` with `schema_version`, `github.repo`, `clickup_list_id`, `slack_channel`, and `skills.plugins`. No `aws[]` needed — the repo has no AWS footprint. |

## Prioritized recommendations

1. **[S] Partial — required CI checks (item 16, RED critical gate):** Add a repo-level ruleset on `main` requiring the Tests matrix and static-analysis checks to pass before merge; the workflows already run on every PR.
2. **[S] Gap — agent-dispatch manifest (item 49):** Add `.agents/pattern-agents.json` with core fields (schema version, GitHub repo, ClickUp list, Slack channel, skills plugins); omit `aws[]` (no AWS footprint).
3. **[S] Gap — CODEOWNERS (item 9):** Add `.github/CODEOWNERS` routing all paths to the `ai-infra-comms` team so the org ruleset's required review reaches the owning team.
4. **[M] Gap — coverage thresholds (item 29):** Run `pytest --cov` with a ratcheted minimum on the ubuntu CI leg; prefer contributing this upstream to avoid workflow drift on fork syncs.
5. **[M] Gap — license compliance scanning (item 18):** Add a periodic dependency-license scan, ideally as an org-side (Wiz policy) control rather than repo CI.
6. **[M] Gap — import boundary enforcement (item 22):** Encode the export/import rules documented in `CLAUDE.md` as ruff banned-import or import-linter contracts; propose upstream first.
7. **[M] Gap — devcontainer (item 36):** Optionally add a `.devcontainer/` wrapping `uv sync` for uniform environments; low urgency given uv pinning.
8. **[L] Gap — mutation testing (item 30):** Scheduled, scoped `mutmut` run over core server/tool modules to validate test strength.
9. **[S] Partial — ADRs (item 3):** Start a dated `docs/adr/` log, seeded from the `v3-notes/` design docs, and record any Pattern-local divergence decisions there.
10. **[S] Partial — runbooks (item 4):** Write the fork-maintenance runbook: upstream sync cadence, local-commit carry strategy, conflict handling.
11. **[S] Partial — complexity limits (item 21):** Make the loq pre-commit hook enforcing (it currently warns), or enable ruff `C901`.
12. **[S] Partial — flaky quarantine (item 32):** Document a quarantine policy using the existing pytest-retry/flakefinder tooling and `client_process` isolation pattern.
13. **[S] Partial — structured CI output (item 33):** Emit and upload `--junitxml` from `.github/actions/run-pytest` so failures are machine-readable.
14. **[M] Partial — commit conventions (item 14):** Enforce the documented PR-label convention (label-check action) or adopt a PR-title lint.
15. **[M] Partial — benchmarks (item 31):** Extend `scripts/benchmark_imports.py` with a small request-dispatch benchmark suite if performance regressions become a concern.
16. **[M] Partial — structured logging (item 43):** Offer an optional JSON log handler in `fastmcp.utilities.logging`; propose upstream.

## Declined practices

| # | Practice | Rationale |
|---|----------|-----------|
| 8 | On-call playbooks | Library fork; nothing operated in production from this repo — no service, no incidents, no pager. |
| 17 | Dependency allow/deny lists | Tracking fork must follow upstream's dependency set; a local policy would block every sync. Dependency policy applies where Pattern consumes the package. |
| 27 | End-to-end tests (Playwright) | No owned browser UI; client-server round-trip and integration tests cover the full-flow surface. |
| 28 | Visual regression tests | No owned visual surface; app UIs render inside MCP host clients. |
| 35 | Smoke tests for deploys | No deployment from this repo; publishing is an upstream release action. |
| 38 | Seed scripts for local databases | No database. |
| 40 | Scoped secrets per environment | No environments or owned credentials; the only credentialed workflow uses OIDC trusted publishing with no stored secrets. |
| 41 | Preview environments per PR | Library; nothing to deploy per PR. |
| 46 | Database migration tooling | No database. |

## Beyond the checklist

- Exceptionally agent-ready documentation: `CLAUDE.md` (symlinked as `AGENTS.md`) encodes workflow, PR/commit norms, module-export philosophy, and critical patterns, mirrored by `.github/copilot-instructions.md` and `.cursor/rules/core-mcp-objects.mdc` for other agent tools.
- Heavy CI automation for maintainer leverage: Marvin/Martian workflows for issue dedupe, triage labeling, MRE enforcement, PR commenting, and test-failure analysis (`.github/workflows/marvin-*.yml`, `martian-*.yml`), plus CodeRabbit configuration.
- `.ccignore` scopes agent context away from generated and vendored paths (`docs/python-sdk/`, `uv.lock`-adjacent noise).
- CI tests against lowest-direct dependency resolutions (`run_tests_lowest_direct`), catching version-floor breakage most libraries miss.
- loq file-size ratcheting (`loq.toml`) is a novel guard against unbounded module growth.
- Supply-chain-conscious publishing: PyPI trusted publishing (OIDC) instead of long-lived tokens.
- Backstage onboarding (`backstage.yaml`) registers the fork in Pattern's service catalog with a named owning team.
