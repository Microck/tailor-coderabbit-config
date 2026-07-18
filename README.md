# tailor-coderabbit-config

A Codex skill that builds and calibrates a repository's `.coderabbit.yaml` for [CodeRabbit](https://coderabbit.ai) review. It treats config as an engineering artifact: start with the smallest setting justified by the repository, then tune it against real reviews. No copied presets.

## Branch modes

- **Bootstrap** - create `.coderabbit.yaml` where none exists.
- **Audit** - check an existing file against the repo and live schema, no changes unless asked.
- **Tune** - address observed misses, noise, or workflow friction.
- **Operate** - inspect resolved config or run a review after the config is ready.

## How it works

Each run fetches the [live schema](https://coderabbit.ai/integrations/schema.v2.json), maps the repository (languages, trust boundaries, CI, linters, guideline docs, review history), and builds an evidence ledger so every non-default setting has to earn its place. It drafts the minimum config, validates with `check-jsonschema`, then loops over representative reviews classifying results as signal, duplicate, false positive, miss, or noise. Mechanism selection, the path-instruction test, and the custom-check test live in [`references/mechanism-selection.md`](references/mechanism-selection.md).

## Install

```bash
git clone https://github.com/Microck/tailor-coderabbit-config.git \
  ~/.codex/skills/tailor-coderabbit-config
```

## Invoke

```
Use $tailor-coderabbit-config to tune this repository's .coderabbit.yaml.
```

## Maturity

- **Baseline** - mapped and schema-valid, untested against real reviews.
- **Calibrated** - representative reviews triaged, repeated patterns handled.
- **Mature** - two clean review cycles, blocking checks proven in warning mode first.

## Notes

Live PR interaction (`@coderabbitai configuration`, `@coderabbitai review`) is opt-in and requires explicit permission on third-party repos. The skill never merges, closes, or touches unrelated PR state.
