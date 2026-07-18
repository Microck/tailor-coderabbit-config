---
name: tailor-coderabbit-config
description: Tailor, validate, and calibrate a repository's .coderabbit.yaml from codebase evidence and real review feedback. Use when creating, auditing, optimizing, or iterating CodeRabbit configuration, path instructions, path filters, code guidelines, review tools, auto-review behavior, inheritance, or pre-merge checks for a specific repository.
---

# Tailor CodeRabbit Config

Build a calibration, not a preset. Start with the smallest configuration justified by this repository, then tune it from actual CodeRabbit behavior.

## Select the branch

- **Bootstrap**: Create `.coderabbit.yaml` where none exists.
- **Audit**: Check an existing file against the repository and current schema without changing it unless asked.
- **Tune**: Change an existing file to address observed misses, noise, or workflow friction.
- **Operate**: Inspect resolved configuration or run a review after the config is ready and the user authorized live PR interaction.

For every branch, read [mechanism-selection.md](references/mechanism-selection.md) before drafting.

## 1. Establish authority and ground truth

1. Read the repository's applicable agent instructions and development contract.
2. Inspect version-control status before editing. Preserve unrelated work.
3. Locate the existing `.coderabbit.yaml`, central configuration clues, CodeRabbit comments/checks, and repository ownership.
4. Fetch the current schema rather than trusting remembered fields or copied examples:

```bash
curl -L -sS -o /tmp/coderabbit-schema.v2.json \
  https://coderabbit.ai/integrations/schema.v2.json
```

5. When a PR exists, inspect existing CodeRabbit reviews and configuration comments read-only. Post `@coderabbitai configuration` only when live PR interaction is in scope. On a third-party repository, require explicit permission before any CodeRabbit trigger.
6. Treat the resolved configuration as the runtime truth. Repository YAML can be changed by central config, UI settings, inheritance, and global overrides.

Completion criterion: the active repository, applicable instructions, current config sources, ownership, and live schema are known, or each unavailable item is explicitly reported.

## 2. Build the review map

Inspect enough code and configuration to account for every material subsystem. Use repository-native structural search for definitions and dependencies, and exact search for config and literal paths.

Record:

- languages, frameworks, package managers, and workspace boundaries;
- runtime entry points, public APIs, protocols, persistence, migrations, and generated code;
- trust boundaries such as authentication, authorization, secrets, payments, and external input;
- test layout, CI gates, linters, formatters, security scanners, and their config files;
- repository instructions such as `AGENTS.md`, `CLAUDE.md`, Copilot instructions, and coding standards;
- release-note, documentation, and PR-title conventions;
- bot PRs, draft workflow, target branches, and expected review cadence;
- historical CodeRabbit comments, especially repeated misses and false positives.

Do not infer a convention from one file when repository-level enforcement or representative examples are available.

Completion criterion: every material subsystem and changed-file family has an owner, an existing enforcement path, or an identified review gap.

## 3. Create the evidence ledger

Before writing YAML, produce this compact ledger in chat or a user-requested artifact:

| Concern | Repository evidence | Existing enforcement | CodeRabbit mechanism | Decision |
|---|---|---|---|---|
| Example: SQL migrations | migration directory and runbook | CI syntax check only | custom check in warning mode | add |

Use CodeRabbit only for a real gap. Prefer the existing linter, CI test, or guideline document when it already enforces the concern reliably. Do not duplicate prose across global tone, `**/*` instructions, path instructions, and custom checks.

Completion criterion: every proposed non-default setting has repository evidence and one reason it belongs in CodeRabbit rather than existing enforcement.

## 4. Draft the minimum config

Start with the schema directive:

```yaml
# yaml-language-server: $schema=https://coderabbit.ai/integrations/schema.v2.json
```

Then apply these rules:

1. Omit schema defaults unless the explicit value documents an intentional policy or protects against inherited settings.
2. Set `inheritance` only after resolving the actual organization or central-config behavior.
3. Use `path_filters` only for repository-specific generated, vendored, or irrelevant paths not already covered by CodeRabbit defaults. Confirm each glob against real paths.
4. Use `path_instructions` for concrete, path-scoped review gaps. State observable failure conditions and repository-specific context. Avoid generic `**/*` advice that restates normal code review.
5. Let CodeRabbit auto-detect standard guideline files. Add `knowledge_base.code_guidelines.filePatterns` only for additional standards files that truly apply as review criteria.
6. Configure a review tool only when the repository contains relevant files and any required tool config. Do not enable every available tool for completeness.
7. Use custom pre-merge checks for deterministic pass/fail repository invariants that CodeRabbit can inspect in its read-only sandbox. Start new checks in `warning`; promote to `error` only after calibration and only when request-changes behavior is understood.
8. Match `auto_review` to the repository's real PR workflow. Exact bot usernames, labels, draft policy, branch regexes, and commit cadence must come from evidence.
9. Keep `tone_instructions` about communication style. Put review criteria in the narrowest applicable mechanism.
10. Preserve useful existing settings unless evidence shows they are stale, duplicated, or harmful.

Completion criterion: the YAML contains only evidence-backed settings, every glob is deliberate, and no instruction has a narrower or more deterministic home.

## 5. Validate mechanically and semantically

Run live-schema validation without adding dependencies to the repository:

```bash
uvx --from check-jsonschema==0.37.4 check-jsonschema \
  --schemafile https://coderabbit.ai/integrations/schema.v2.json \
  .coderabbit.yaml
```

If `uvx` is unavailable, use the repository's existing JSON-Schema-capable YAML validator or the official CodeRabbit YAML validator. Report any weaker validation path.

Then audit semantics:

- `.coderabbit.yaml` is at the repository root;
- every configured path, config file, username, label, and branch is verified;
- instructions agree with repository contracts and do not duplicate stronger enforcement;
- custom checks use information available in CodeRabbit's sandbox;
- blocking modes cannot surprise the repository's merge workflow;
- the diff contains no unrelated changes.

When live PR operation is authorized, request `@coderabbitai configuration` and compare the resolved result with intent. Do not treat local schema success as proof that inherited runtime behavior matches.

Completion criterion: syntax and live schema pass, semantic checks pass, and resolved runtime configuration is either verified or clearly marked unverified.

## 6. Run the calibration loop

For Tune or Operate work, inspect 3-5 representative reviewed PRs, or every available review when only 1-2 exist. With no review history, remain at Baseline. Classify each relevant CodeRabbit result:

- **signal**: correct, novel, and actionable;
- **duplicate**: already enforced or already reported elsewhere;
- **false positive**: wrong because repository context was missing;
- **miss**: a repository invariant should have been caught;
- **workflow noise**: timing, scope, summaries, or repeated incremental reviews are the problem.

Change the smallest mechanism that addresses the repeated pattern:

- narrow or remove an instruction for false positives;
- remove duplication when CI or a linter is authoritative;
- add one scoped instruction or warning check for a miss;
- tune `auto_review` for workflow noise;
- prefer repository guideline files when the same rule should guide both coding agents and reviews.

Validate after every change. If the user authorized a live test, request a fresh review only when no equivalent review is already running for the current head. Never merge, close, or modify unrelated PR state.

Completion criterion: every sampled repeated pattern is classified and addressed or explicitly accepted, the final config validates, and the maturity label below is honest.

## 7. Report maturity

Use one label:

- **Baseline**: repository-mapped and schema-valid, but not yet tested against representative CodeRabbit reviews.
- **Calibrated**: at least three representative reviews, or every available review when only 1-2 exist, were triaged and every repeated config-driven pattern was handled.
- **Mature**: two subsequent review cycles across affected paths show no recurring config-driven false positives, and blocking checks have proven reliable in warning mode first.

Never call a one-pass configuration perfect. State what evidence would move it to the next label.

Finish with changed files, intentionally untouched areas, risks, validation commands/results, runtime verification status, and the maturity label. If editing left newly unused config or rule files, list them and ask before removal.
