# Migration from historical workflows

The active workflows use one readable CI entry point for each project type.
First-generation workflows remain under `.github/workflows/historical/` as a
reference, but GitHub does not load them from that nested directory.

Update every caller before relying on one of the historical workflow paths.

## Workflow mapping

| Historical workflow | Replacement |
|---|---|
| `lesli-ci-tests.yaml` | `lesli-ci-app.yaml` for a host, or `lesli-ci-engine.yaml` for an engine |
| `lesli-test-engine-brakeman.yaml` | Security stage inside `lesli-ci-app.yaml` or `lesli-ci-engine.yaml` |
| `lesli-test-engine-minitest.yaml` | Test stage inside `lesli-ci-engine.yaml` |
| `lesli-test-gem-minitest.yaml` | `lesli-ci-gem.yaml` |
| `lesli-test-builder.yaml` | `lesli-test-integration.yaml` |
| `lesli-test-new.yaml` | `lesli-test-integration-new.yaml` |

## Recommended migration

1. Classify the repository as a Rails app, Rails engine/Rails gem, or standalone
   Ruby gem.
2. Replace multiple reusable-workflow jobs with the corresponding single
   `lesli-ci-*` job.
3. Remove the old `branch` input. Integration workflows now test the exact
   caller commit automatically.
4. Set `module-directory` to `engines` or `gems` for Rails modules.
5. Map `CODECOV_TOKEN` only when the repository uploads to Codecov.
6. Run CI with the `@master` reference during adoption.
7. Move to `@v1` after the stable tag is published.

## Engine example

An engine previously needed separate jobs for Brakeman, Minitest, and host
integration. Replace them with:

```yaml
jobs:
  ci:
    uses: LesliTech/LesliActions/.github/workflows/lesli-ci-engine.yaml@master
    with:
      module-directory: "engines"
      security-path: "test/dummy"
      run-security: true
      run-integration: true
      run-integration-new: false
    secrets:
      CODECOV_TOKEN: ${{ secrets.CODECOV_TOKEN }}
```

The main workflow enforces the stage order. There is no need to recreate
cross-job dependencies in every engine repository.

## Rails gem example

For a gem that implements `Rails::Engine` and is installed under `gems/`:

```yaml
jobs:
  ci:
    uses: LesliTech/LesliActions/.github/workflows/lesli-ci-engine.yaml@master
    with:
      module-directory: "gems"
      run-security: false
      run-integration: true
      run-integration-new: false
    secrets:
      CODECOV_TOKEN: ${{ secrets.CODECOV_TOKEN }}
```

## Standalone gem example

For a gem without Rails:

```yaml
jobs:
  ci:
    uses: LesliTech/LesliActions/.github/workflows/lesli-ci-gem.yaml@master
    with:
      ruby-version: "3.3.5"
    secrets:
      CODECOV_TOKEN: ${{ secrets.CODECOV_TOKEN }}
```

## Validation checklist

Before merging a migrated caller, verify:

- The repository exposes the required test and build commands.
- `COVERAGE=true` generates `coverage/coverage.xml`.
- `bundle exec rake build` creates `pkg/*.gem` for engines and gems.
- Brakeman is bundled and `security-path` is correct when security is enabled.
- The integration module directory matches the LesliBuilder layout.
- Required refs point to compatible LesliBuilder, Lesli, and LesliActions code.
- The workflow has no references to files inside `historical/`.
