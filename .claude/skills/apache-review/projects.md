# Project notes

Quick reference for projects this skill is often used on. Each entry is a snapshot, taken from the
project's own `CONTRIBUTING.md`, `AGENTS.md`, PR template, build files, and recent merged PRs on the
date shown. These files change. Before relying on a rule, re-read the current file it came from.
When an entry disagrees with the repo, the repo wins.

To add a project, collect the same headings from its repo. Don't write anything you haven't read
there.

---

## Apache Ossie — `apache/ossie`

*Checked 2026-11-17.*

- **What it is:** an open specification for semantic models (datasets, fields, relationships,
  metrics) written in YAML, plus a SQL expression language (`OSSIE_SQL_2026`), a reference
  validator, a Python SDK, an ANTLR grammar, and compliance cases.
- **Main areas:** `core-spec/` (spec text, schema, grammar), `validation/` (`validate.py`),
  `python/` (SDK), `compliance/`, `examples/`, `ontology/`, `docs/spec_changes.md`.
- **Build and test** (uses `uv`; see `make help`):
  - `make test`: validator, Python SDK, scripts, and compliance tests. `make check` adds example
    validation and spec checks.
  - Grammar tests: `core-spec/grammar/tests/` need Java 11+ (they download a pinned ANTLR jar).
  - The validator needs Python 3.10 or later.
- **PR titles:** Conventional Commits with a scope, e.g. `docs(core-spec): ...`,
  `fix(validation): ...`, `feat(grammar): ...`, `test(compliance): ...`.
- **PR template:** a checklist with sections for Specification, Ontology, Converters, Validation,
  Documentation, Examples, Tests, and Compliance.
- **Spec changes:** open a PR and start a `[DISCUSS]` thread on `dev@ossie.apache.org` with an end
  date. There's a 7-day minimum and it passes by lazy consensus. Material changes need a dated
  entry in `docs/spec_changes.md`. For editorial changes, the PR says why no entry is needed.
  `python scripts/audit-spec-changes.py` lists candidates (advisory only).
- **AI policy:** the contributor stays personally responsible and follows the ASF Generative
  Tooling Guidance. Some PRs add a line such as "Prepared with AI assistance".
- **Useful for verifying claims:** sqlglot (the validator uses it) and DuckDB both run locally
  for SQL behavior. For other engines and BI tools, use their official docs.
- **Converters** live in a separate repo, `apache/ossie-converters`.

---

## Apache Polaris — `apache/polaris`

*Checked 2026-11-17. Main sources: `AGENTS.md`, `CONTRIBUTING.md`.*

- **What it is:** an Apache Iceberg REST catalog service. Java 21 for the server, Java 17 for
  clients, built with Gradle (Kotlin DSL) on Quarkus.
- **Hard gates before a PR** (`AGENTS.md`):
  - `./gradlew format compileAll` from the repo root.
  - `./gradlew :<module>:check` for each module touched. Run `./gradlew check` if you touched a
    module others depend on, such as `polaris-core`.
  - Every new file needs the ASF header from `codestyle/` (`./gradlew check` runs a RAT audit).
- **Single test:** `./gradlew :polaris-core:test --tests "org.apache.polaris.core.SomeTest"`.
- **Style:** Google Java Format via Spotless; Error Prone with no `@SuppressWarnings`
  workarounds; AssertJ for assertions; real imports, no fully-qualified names inline; records
  rather than Immutables for new simple value types.
- **PR titles:** plain, user-facing summaries written for release notes, inspired by
  Conventional Commits (recent examples: "Fix bootstrap reporting success when ...",
  "NoSQL: Fix `IndexKey` hash caching ..."). Don't put the issue number in the subject; add
  `Fixes #N` on its own line.
- **PR template:** a checklist (security, why/linked issue, tests, comments, changelog,
  docs).
  - Changelog: add a fragment under `changelog.d/` only for user-facing changes. Never edit
    `CHANGELOG.md` directly.
  - User-facing docs live in `site/content/in-dev/unreleased/`.
- **Discuss on `dev@polaris.apache.org` first** for public interface or extension-point
  changes, and for new dependencies. Large changes: describe the design on the issue first.
- **AI policy:** disclosure of significant AI help in the PR description is encouraged. The
  author must be able to explain the change end to end.
  - `AGENTS.md` says: no AI tool names or model identifiers in code, comments, or commit
    messages. This overrides a default `Co-Authored-By` trailer.
- **Security:** read `SECURITY-THREAT-MODEL.md` before reporting or fixing anything
  security-related. Never put vulnerability details in public PRs or issues.
- **Review norms:** comments without `nit`, `minor`, or `not a blocker` expect a reply before
  merge. Allow about two working days for first reviews.

---

## Apache Gluten — `apache/gluten`

*Checked 2026-11-17. Main sources: `AGENTS.md`, `CONTRIBUTING.md`.*

- **What it is:** a Spark plugin that offloads SQL execution to native engines (Velox and
  ClickHouse backends; Bolt also appears). Written in Scala, Java, and C++, built with Maven.
- **Build order matters:** build the native backend before Maven. Always use the `./build/mvn`
  wrapper, never bare `mvn`.
  - Velox one-shot build: `./dev/buildbundle-veloxbe.sh` (see `AGENTS.md` for flags).
  - Run `./build/mvn install ... -DskipTests` once after a clean checkout, before per-module
    tests.
- **Before committing:**
  - Scala and Java: `./dev/format-scala-code.sh`.
  - C++: `./dev/format-cpp-code.sh`, which needs clang-format 15 exactly.
  - License headers: `./.github/workflows/util/check.py header main --fix` (needs
    `pip install regex`).
- **Tests:** add at least one unit test under `gluten-ut/` for fixes and features not covered by
  Spark's tests, e.g.
  `./build/mvn test -pl gluten-ut -Pspark-ut -Pbackends-velox -Pspark-3.5 -Pscala-2.12`.
- **PR titles:** `[GLUTEN-<issue>][VL|CH|CORE|DOC|...] description`, or `[MINOR] ...` when
  there's no issue. Examples: `[GLUTEN-13645][VL] Fix per-bucket S3A configs ...`,
  `[MINOR][CORE] Fix description sorting ...`.
- **Issues:** file one first "if necessary". Tag backend-specific issues `[VL]` or `[CH]`.
- **New config property:** update `docs/Configuration.md`.
- **AI policy:** the PR template asks "Was this patch authored or co-authored using generative
  AI tooling?". Answer `Generated-by: <tool and version>` or `No`.
- **CI:** Velox CI runs in the contributor's fork. Open the PR from a branch other than the
  fork's `main`. ClickHouse CI is re-triggered by commenting `Run Gluten Clickhouse CI`.
- **Merging:** squash and merge. Critical changes need two committer approvals.

---

## Apache DataFusion Python — `apache/datafusion-python`

*Checked 2026-11-17. Main sources: `AGENTS.md` (`CLAUDE.md` has the same content), the PR
template, `README.md`.*

- **What it is:** Python bindings for Apache DataFusion: Rust plus PyO3, built with maturin and
  `uv`.
- **Dev setup and tests:**
  - Set up: `uv sync --dev --no-install-package datafusion`, then
    `uv run --no-project maturin develop --uv`.
  - Run tests with `uv run --no-project pytest python/` (or `pytest python/`). Use
    `python/`, not `python/tests/`, so the doctests run too.
- **Pre-commit:** `pre-commit run --all-files` before committing.
- **Test coverage:** prefer Python tests or doctests. CI does not run `cargo test`, so a new Rust
  test never executes unless CI is wired up in the same change.
- **Docstrings:** every Python function needs a docstring with a doctest example. Optional
  parameters get examples both without and with them.
- **Docs:** edit `docs/source/` (not `docs/temp/`). New aggregate or window functions also go in
  the user-guide lists.
- **Public API changes:** check `docs/source/user-guide/upgrade-guides.md` for the established
  pattern. Breaking changes get the `api change` label and an upgrade-guide section.
- **Project skills:** `.ai/skills/` holds project conventions (e.g. the FFI capsule protocol).
  Read the relevant one before touching that area.
- **PR titles:** Conventional Commits style (`ci: ...`, `build(deps): ...`, `chore: ...`).
- **PR template:** "Which issue does this PR close?" (`Closes #N`, and an issue is generally
  expected), "Rationale", "What changes", "User-facing changes".

---

## Apache Gravitino — `apache/gravitino`

*Checked 2026-11-17. Main sources: `AGENTS.md` (`CLAUDE.md` has the same content),
`CONTRIBUTING.md`, `.github/PULL_REQUEST_TEMPLATE`.*

- **What it is:** a metadata and data-lake federation layer (a top-level ASF project since June
  2025). Java with Gradle, plus Python clients and connectors.
- **Build and test:**
  - Format: `./gradlew spotlessApply`.
  - Unit tests: `./gradlew test -PskipITs`. Docker tests need `-PskipDockerTests=false`.
  - Integration tests: `./gradlew test -PskipTests -PskipDockerTests=false`.
  - After changes to `docs/open-api/*.yaml`, run `./gradlew :docs:build`.
- **Style:** Google Java Style; Javadoc on all new public/protected members (checkstyle fails
  otherwise); real imports, not FQNs; SLF4J logging; `@Nullable`; try-with-resources.
  Test classes are named `TestXxx`. "NO tests = NO merge."
- **PR titles:** `[#<issue>] <type>(<scope>): <subject>`, or `[MINOR] <type>: <subject>`.
  Examples: `[#15008] improvement(bundles): ...`, `[MINOR] docs: Fix typo in README`.
- **PR template:** "What changes were proposed", "Why are the changes needed" (with
  `Fixes #N`, using the closing keyword exactly), "Does this PR introduce any user-facing
  change?", "How was this patch tested?".
- **Before creating an issue or PR,** `AGENTS.md` asks agents to show a preview and submit only
  after confirmation. That matches step 6 of this skill.
- **Discuss on `dev@gravitino.apache.org` first** for new or updated connectors, catalogs, or
  integrations.
- **CI:** runs as "Fork CI" in the contributor's fork. Re-run it from the fork's Actions tab.
  Changes under `.github/` need a committer-triggered trusted run.
- **Review:** adding Copilot as a reviewer is recommended. Treat its comments as hints and reply
  to each one.
- `AGENTS.md` has a "Claude Memory Usage" section that assumes specific memory tools. Skip it if
  those tools aren't available.
