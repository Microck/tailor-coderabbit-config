# tailor-coderabbit-config

A Codex skill that builds and calibrates a repository's CodeRabbit YAML or TypeScript configuration for [CodeRabbit](https://coderabbit.ai) review. It treats config as an engineering artifact: start with the smallest setting justified by the repository, then tune it against real reviews. No copied presets.

## Branch modes

- **Bootstrap** - create a config only after checking all supported formats and effective inherited policy.
- **Audit** - check an existing file against the repo and live schema, no changes unless asked.
- **Tune** - address observed misses, noise, or workflow friction.
- **Operate** - inspect resolved config or run a review after the config is ready.

## How it works

Each run fetches the [live schema](https://coderabbit.ai/integrations/schema.v2.json), maps the repository (languages, trust boundaries, CI, linters, guideline docs, review history), and builds an evidence ledger so every non-default setting has to earn its place. It drafts the minimum config, validates YAML against the live schema or TypeScript through the supported compiler and evaluated output, then, for Tune and Operate, loops over representative reviews classifying results as signal, duplicate, false positive, miss, or noise. Bootstrap and Audit can finish after validation; repositories without review history remain at Baseline. Mechanism selection, the path-instruction test, and the custom-check test live in [`references/mechanism-selection.md`](references/mechanism-selection.md).

## Install

```bash
git clone https://github.com/Microck/tailor-coderabbit-config.git \
  ~/.codex/skills/tailor-coderabbit-config
```

## Invoke

```
Use $tailor-coderabbit-config to tune this repository's CodeRabbit configuration.
```

## Maturity

- **Baseline** - mapped and schema-valid, untested against real reviews.
- **Calibrated** - representative reviews triaged, repeated patterns handled.
- **Mature** - two clean review cycles, blocking checks proven in warning mode first.

## Notes

Live PR interaction (`@coderabbitai configuration`, `@coderabbitai review`) is opt-in and requires explicit permission on third-party repos. The skill does not grant merge authority or touch unrelated PR state. An explicitly authorized caller may merge a scoped configuration PR after its review and checks pass. Requested recurring maintenance updates relevant stale guidance and treats unchanged repositories as no-ops.
