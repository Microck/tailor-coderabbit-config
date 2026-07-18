# tailor-coderabbit-config

A Codex skill that builds, validates, and calibrates a repository's `.coderabbit.yaml` for [CodeRabbit](https://coderabbit.ai) automated code review. Rather than shipping a copied preset, the skill walks an AI coding agent through a calibration workflow grounded in actual codebase evidence and observed CodeRabbit review behavior.

Most CodeRabbit configs start as a copy from a blog post or the awesome-coderabbit list. They carry inherited assumptions, review noise, and gaps that no one ever revisits. This skill treats configuration as an engineering artifact: start with the smallest setting justified by the repository, then tune it against real reviews.

## Why calibration beats a preset

A preset has no knowledge of your repository. It cannot tell that your SQL migrations already pass a CI check, that a path instruction duplicates your linter, or that a custom check can never succeed in CodeRabbit's read-only sandbox. The result is duplicated enforcement, false positives, and blocking checks that surprise the merge workflow.

Calibration inverts the problem. Every non-default setting must earn its place with repository evidence and one reason it belongs in CodeRabbit rather than existing CI, types, or guidelines. Presets say "review for bugs, security, performance, style, and best practices" and spend attention without adding context. A calibrated config says "flag destructive schema changes that lack the documented expand-and-contract sequence," which is something a reviewer can act on.

## Branch modes

The skill opens by asking which mode fits the task, then adapts the workflow:

- **Bootstrap** - create `.coderabbit.yaml` where none exists.
- **Audit** - check an existing file against the repository and current schema without changing it unless asked.
- **Tune** - change an existing file to address observed misses, noise, or workflow friction.
- **Operate** - inspect resolved configuration or run a review after the config is ready and live PR interaction is authorized.

## The calibration workflow

Each phase has an explicit completion criterion so work cannot drift.

**1. Establish ground truth.** Read agent instructions (`AGENTS.md`, `CLAUDE.md`, Copilot instructions), inspect version-control status, locate the existing config, and fetch the current live schema from `https://coderabbit.ai/integrations/schema.v2.json`. Treat the resolved configuration, not the repository YAML, as runtime truth because central config and inheritance can change effective behavior.

**2. Build the review map.** Record languages, frameworks, entry points, public APIs, persistence, trust boundaries, test layout, CI gates, linters, guideline docs, and any historical CodeRabbit comments. Every material subsystem gets an owner, an existing enforcement path, or an identified review gap.

**3. Create the evidence ledger.** Before writing YAML, produce a compact table: concern, repository evidence, existing enforcement, candidate CodeRabbit mechanism, and decision. Use CodeRabbit only for a real gap and never duplicate prose that already lives in a linter or guideline document.

**4. Draft the minimum config.** Start with the schema directive, omit defaults unless they document an intentional policy, confirm every glob against real paths, prefer path-scoped instructions over global advice, start custom checks in warning mode, and match `auto_review` to the real PR workflow with exact bot usernames and labels.

**5. Validate mechanically and semantically.** Run live-schema validation without adding dependencies:

```bash
uvx --from check-jsonschema==0.37.4 check-jsonschema \
  --schemafile https://coderabbit.ai/integrations/schema.v2.json \
  .coderabbit.yaml
```

Then audit semantics: root location, verified paths and usernames, instructions that do not duplicate stronger enforcement, sandbox-safe custom checks, and no unrelated diff.

**6. Run the calibration loop.** For Tune or Operate, inspect three to five representative reviewed PRs (or all available reviews when fewer exist). Classify each CodeRabbit result as signal, duplicate, false positive, miss, or workflow noise, then change the smallest mechanism that addresses the repeated pattern. Validate after every change.

**7. Report maturity.** Close with changed files, untouched areas, risks, validation commands and results, runtime verification status, and an honest maturity label.

## Mechanism selection

`references/mechanism-selection.md` is the decision aid the skill reads before drafting. It maps each need to the narrowest CodeRabbit mechanism and states the evidence required:

| Need | Mechanism |
|---|---|
| Exclude irrelevant changed files | `reviews.path_filters` |
| Guide review for a code area | `reviews.path_instructions` |
| Reuse written engineering standards | auto-detected code guidelines or `filePatterns` |
| Run a supported analyzer | `reviews.tools` |
| Enforce a deterministic PR invariant | custom pre-merge check |
| Control when reviews run | `reviews.auto_review` |
| Change presentation | summary, walkthrough, and tone settings |
| Share organization policy | central config and inheritance |

Every mechanism has a test. A path instruction earns its place only when its glob matches verified paths, the rule is narrower than a repo-wide guideline, it identifies observable risk, existing enforcement does not already cover it, and a reviewer can tell what evidence would make a finding valid. Custom checks get explicit Pass, Fail, and Inconclusive conditions and start in warning mode.

## Installation

This is a Codex skill, not a standalone CLI. Drop the directory into your Codex skills folder and Codex will discover it:

```bash
# Personal skills (available across your projects)
git clone https://github.com/Microck/tailor-coderabbit-config.git \
  ~/.codex/skills/tailor-coderabbit-config

# Or project-local
git clone https://github.com/Microck/tailor-coderabbit-config.git \
  ./.codex/skills/tailor-coderabbit-config
```

The skill ships three files:

- `SKILL.md` - the calibration workflow and branch modes.
- `references/mechanism-selection.md` - mechanism matrix, path-instruction test, custom-check test, and calibration heuristics.
- `agents/openai.yaml` - Codex interface metadata (display name and default prompt).

## Invocation

From an agent session in the target repository, reference the skill by name:

```
Use $tailor-coderabbit-config to tune this repository's .coderabbit.yaml.
```

The skill's default prompt is:

> Use $tailor-coderabbit-config to create or tune this repository's .coderabbit.yaml from codebase evidence and real review feedback.

State the mode if you want a specific path: "bootstrap a new config," "audit the existing one without changes," or "tune after the last review cycle had too many false positives."

## Maturity labels

The skill never calls a one-pass configuration perfect. It reports one label and states the evidence that would move the config to the next.

- **Baseline** - repository-mapped and schema-valid, but not yet tested against representative CodeRabbit reviews.
- **Calibrated** - three representative reviews (or all available when fewer exist) were triaged and every repeated config-driven pattern was handled.
- **Mature** - two subsequent review cycles across affected paths show no recurring config-driven false positives, and blocking checks proved reliable in warning mode first.

## Requirements and notes

- `curl` to fetch the live CodeRabbit schema during every run.
- `uvx` to run `check-jsonschema` without adding a dependency to the target repository. Any JSON-Schema-capable YAML validator is an acceptable fallback, but the skill reports the weaker path.
- Live PR interaction (`@coderabbitai configuration`, `@coderabbitai review`) is opt-in and only when the user authorizes it. On a third-party repository the skill requires explicit permission before any CodeRabbit trigger and never merges, closes, or modifies unrelated PR state.
- The skill validates against the live schema because CodeRabbit fields change over time. Copied examples are candidates only; every key is checked against the current schema and every policy against the target repository.
