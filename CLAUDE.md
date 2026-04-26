# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

This is the CorpDK organization's special **`.github` repository**. It does not contain application code — it provides org-wide GitHub assets that GitHub picks up by convention:

- `profile/README.md` — rendered as the public organization profile page on github.com/CorpDK.
- `.github/ISSUE_TEMPLATE/` — `bug_report.md`, `feature_request.md`, `custom.md`. These templates are inherited as defaults by every repository in the org that does not define its own.
- `.github/workflows/` — **reusable workflows** (`workflow_call`) that other CorpDK repos consume via `uses: CorpDK/.github/.github/workflows/<name>.yml@main`. Changes here ship to every caller on next run, so treat edits as breaking-change-sensitive.
- `LICENSE` — org-default license (referenced by repos without their own LICENSE).
- `README.md` at the repo root is just a placeholder for the repo itself; the user-facing org profile is `profile/README.md`.

When editing, keep this distinction in mind: the file path determines whether GitHub treats the change as the org profile, an inherited issue template, a reusable workflow consumed org-wide, or just internal repo content.

## Reusable workflows

Workflows under `.github/workflows/` are called by other repos in the org. House rules:

- `on: workflow_call` only — these are not meant to run on push/PR in this repo.
- Top-level least-privilege `permissions:` (e.g. `contents: read`, `pull-requests: read`) to satisfy checkov rule **CKV2_GHA_1**. Add scopes only when a step actually needs them.
- Prefer `inputs:` for non-sensitive config (URLs, flags) and `secrets:` only for actual secrets. Mark both `required: true` when the workflow can't function without them.
- Third-party actions: major-version tags are acceptable here (no SHA-pinning policy in this repo). Pin if/when an org policy is introduced.
- `runs-on: ubuntu-latest` unless a workflow has a real reason to differ.
- Repo-specific config (e.g. `sonar-project.properties`, project keys, sources, exclusions) belongs in the **caller** repo, not the reusable workflow. Keep these workflows generic.

Currently published:

- [.github/workflows/sonar.yml](.github/workflows/sonar.yml) — SonarQube/SonarCloud scan. Inputs: `sonar-host-url` (string, required). Secrets: `SONAR_TOKEN` (required). Does `actions/checkout@v4` with `fetch-depth: 0` (full history needed for new-code detection and blame), then `SonarSource/sonarqube-scan-action@v5`.

Operational notes (not enforced by code, easy to forget):

- For other org repos to consume these, `CorpDK/.github` must be **public**, OR Org Settings → Actions → General → Access must allow this repo's workflows to be called by other repos in the org.
- Org-level secrets referenced by callers (e.g. `SONAR_TOKEN`) need visibility to the consuming repos.

## Tooling

[Trunk](https://docs.trunk.io/cli) is the only toolchain configured ([.trunk/trunk.yaml](.trunk/trunk.yaml)). There is no build system, package manager, or test suite.

Enabled linters:

- `markdownlint@0.45.0` — config at [.trunk/configs/.markdownlint.yaml](.trunk/configs/.markdownlint.yaml) extends `markdownlint/style/prettier`, so prettier handles formatting and markdownlint only catches non-formatting issues. Don't re-enable formatting rules in markdownlint — they'll fight prettier.
- `prettier@3.6.2` — formats Markdown and YAML.
- `trufflehog@3.95.2` — secret scanning.
- `git-diff-check` — whitespace/conflict-marker check.

Runtimes pinned: `node@22.16.0`, `python@3.10.8`.

### Common commands

```bash
trunk check          # run all enabled linters on changed files
trunk check --all    # run on the whole repo
trunk fmt            # auto-format (prettier, etc.)
trunk check enable <linter>   # add a linter
trunk upgrade        # bump trunk + plugin versions
```

## Conventions

- Markdown: prettier owns formatting. Run `trunk fmt` before committing rather than hand-tuning line wraps.
- The org profile lives at `profile/README.md` (not the repo's own `README.md`). Edits to the public-facing org page go there.
- Issue templates use Markdown front-matter (`name`, `about`, `title`, `labels`, `assignees`). Preserve the front-matter when editing — GitHub uses it to render the "New issue" picker.
- Reusable workflows: don't add `on: push` / `on: pull_request` triggers — they're consumed via `workflow_call`. Don't introduce repo-specific defaults (project keys, paths, exclusions); push that config back to the caller.
