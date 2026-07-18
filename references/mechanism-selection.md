# CodeRabbit mechanism selection

Read this before drafting or tuning `.coderabbit.yaml`. Fetch the current live schema during every run; this file contains selection rules, not a frozen field catalog.

## Source hierarchy

1. Resolved configuration from `@coderabbitai configuration`
2. Live schema: <https://coderabbit.ai/integrations/schema.v2.json>
3. Current official configuration reference: <https://docs.coderabbit.ai/reference/configuration>
4. Repository contracts, code, CI, and review history
5. Official examples: <https://docs.coderabbit.ai/configuration/example>
6. Community configurations, including <https://github.com/coderabbitai/awesome-coderabbit>

Use examples to discover candidates only. Validate every copied key against the live schema and every copied policy against the target repository.

## Mechanism matrix

| Need | Mechanism | Evidence required | Avoid |
|---|---|---|---|
| Exclude irrelevant changed files | `reviews.path_filters` | actual generated/vendor paths beyond defaults | broad exclusions that hide reviewable code |
| Guide review for a code area | `reviews.path_instructions` | path-specific invariant or observed miss | generic review advice or style rules already linted |
| Reuse written engineering standards | auto-detected code guidelines or `filePatterns` | authoritative instruction document and correct scope | copying the same prose into YAML |
| Run a supported analyzer | `reviews.tools` | relevant source files and required config | enabling tools only because they exist |
| Enforce a deterministic PR invariant | custom pre-merge check | explicit pass/fail rule inspectable read-only | tests, builds, approvals, or subjective quality judgments |
| Control when reviews run | `reviews.auto_review` | actual labels, bot names, branch targets, and cadence | guessed usernames or inherited workflow assumptions |
| Change presentation | summary, walkthrough, and tone settings | team preference or observed noise | mixing product policy into tone |
| Share organization policy | central config and inheritance | known hierarchy and resolved source | setting `inheritance: true` by habit |
| Supply cross-repository context | linked repositories | concrete dependency and access | linking loosely related repositories |

## Path instruction test

A path instruction earns its place only when all are true:

1. Its glob matches verified repository paths.
2. The rule is narrower than a repository-wide guideline.
3. The rule identifies observable risk, not a vague quality preference.
4. Existing CI, linters, and types do not already enforce it reliably.
5. A reviewer can tell what evidence would make a finding valid.

Prefer:

```yaml
reviews:
  path_instructions:
    - path: "db/migrations/**"
      instructions: |
        Flag destructive schema changes that lack the repository's documented
        expand-and-contract sequence. Cite the changed migration and the missing
        phase. Do not flag additive nullable columns.
```

Avoid global prompts such as "review for bugs, security, performance, style, and best practices." They spend attention without adding repository context.

## Custom check test

Use a custom check only when CodeRabbit's sandbox can decide from changed files, repository text, git history, PR metadata, linked issues, or configured MCP context. It cannot run the test suite, install dependencies, execute arbitrary repository code, inspect approval state, or post inline comments.

Write one concern per check with explicit Pass, Fail, and Inconclusive conditions. Start in `warning`. Move to `error` only after representative warnings are correct and the repository intentionally uses the request-changes workflow.

Official behavior and limitations: <https://docs.coderabbit.ai/pr-reviews/custom-checks>

## Defaults and inheritance

Omitting a field accepts the active default or inherited value. Explicitly setting a default can still be justified when it is a deliberate local override, but record that reason in the evidence ledger.

Inheritance is disabled by default at a configuration level, but organization and workspace global overrides can change the effective chain. Objects deep-merge, arrays merge and deduplicate by stable keys, and scalars override. Verify the resolved configuration rather than reasoning from the repository YAML alone.

Official inheritance behavior: <https://docs.coderabbit.ai/configuration/configuration-inheritance>

## Calibration heuristics

- Remove a rule after repeated false positives before adding exceptions around it.
- Narrow a rule by path before lengthening its prompt.
- Prefer a deterministic analyzer over natural-language duplication.
- Treat a one-off miss as evidence to investigate, not automatic permission to add a rule.
- Promote a warning to an error only after it is both accurate and operationally understood.
- Revisit auto-review cadence when comments are useful but arrive too often.
- Use `@coderabbitai emit path instructions` as candidate generation after several reviews, then audit every suggestion before accepting it.

Official path guidance: <https://docs.coderabbit.ai/configuration/path-instructions>
Official auto-review controls: <https://docs.coderabbit.ai/configuration/auto-review>
Official YAML setup: <https://docs.coderabbit.ai/getting-started/yaml-configuration>
