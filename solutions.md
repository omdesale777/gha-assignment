# Assignment Notes / Bug Table

Replace each `RUN_URL_*` placeholder with the actual successful GitHub Actions run URL after running the workflows.

## Q3 — This workflow never even starts

| Bug | Cause | Fix |
|---|---|---|
| `workflow_dispatch` was nested under `push` / incorrectly structured | `on` must define `push` and `workflow_dispatch` as sibling triggers | Put `workflow_dispatch:` at the same level as `push:` |
| Invalid runner label `ubuntu-latest-large` | Standard GitHub-hosted runners use labels such as `ubuntu-latest` and `ubuntu-22.04`; the made-up large label is not a standard hosted runner label | Changed to `ubuntu-latest` |
| Script ran before checkout | A fresh GitHub-hosted runner does not contain the repository files until checkout happens | Moved `actions/checkout` before the script step |
| Checkout action reference was incomplete | `uses` must identify an action ref such as `actions/checkout@v7` | Added `@v7` |

## Q4 — The deploy job is always skipped

| Bug | Cause | Fix |
|---|---|---|
| Input accepted arbitrary text | Free-text input is error-prone for a fixed set of environments | Changed to `type: choice` with `staging` and `production` |
| Job-level condition skipped staging | `if: inputs.environment == production` on the whole job prevents the job from running for staging | Removed job-level condition |
| Production-only behavior was attached to the wrong scope | The requirement is to run the job for both values but print the production line only for production | Added step-level `if: ${{ inputs.environment == 'production' }}` |
| Guarded step used shell-style `$APP_NAME` inside an expression | GitHub expressions use contexts such as `env.APP_NAME`, not shell `$VAR` syntax | Changed to `if: ${{ env.APP_NAME == 'demo-app' }}` |
| Notification should run even after a failed earlier step | Default step condition is success-only | Changed notify to `if: ${{ always() }}` |

## Q5 — Job outputs come through empty

| Bug | Cause | Fix |
|---|---|---|
| `::set-output` in `make` | GitHub deprecated the `saveState`/`set-output` workflow commands; step outputs should now be written to `$GITHUB_OUTPUT` | Changed to `echo "version=..." >> "$GITHUB_OUTPUT"` |
| Consumer job had no dependency on generator | Without `needs: generate`, GitHub does not establish the job dependency/order needed for `needs.generate.outputs...` | Added `needs: generate` |
| Output must be consumed through the job-output path | Step outputs become job outputs only through `jobs.<job>.outputs`, then another job reads them through `needs.<job>.outputs.<name>` | Kept `generate.outputs.version` mapped from `steps.make.outputs.version`, and consumed via `needs.generate.outputs.version` |

### Why did GitHub deprecate `::set-output`?

GitHub moved workflow command handling away from stdout control commands and toward environment files. The environment-file approach (`$GITHUB_OUTPUT`, `$GITHUB_ENV`, etc.) is safer and less ambiguous because commands are communicated through dedicated files instead of being parsed from ordinary log output. For step outputs specifically, GitHub's current documentation shows writing `name=value` to `$GITHUB_OUTPUT`.

## Run links

- Q1: RUN_URL_Q1
- Q2: RUN_URL_Q2
- Q3: RUN_URL_Q3
- Q4: RUN_URL_Q4
- Q5: RUN_URL_Q5
- Q6: RUN_URL_Q6
