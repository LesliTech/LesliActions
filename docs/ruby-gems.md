# Standalone Ruby gems

Use `lesli-ci-gem.yaml` for Ruby gems that do not require Rails. Examples include
Termline and LesliTesting.

## Requirements

The gem must provide:

```shell
bundle exec rake test
bundle exec rake build
```

The test task runs with `COVERAGE=true` and must generate
`coverage/coverage.xml`. The build task must generate at least one `pkg/*.gem`
file.

The workflow intentionally does not load Rails, prepare a database, run
Brakeman, or assemble a Lesli host.

## Caller workflow

Create `.github/workflows/lesli-ci.yaml` in the gem repository:

```yaml
name: "Lesli CI"

on:
  push:
    branches: [master]
  pull_request:
  workflow_dispatch:

jobs:
  ci:
    uses: LesliTech/LesliActions/.github/workflows/lesli-ci-gem.yaml@master
    with:
      ruby-version: "3.3.5"
    secrets:
      CODECOV_TOKEN: ${{ secrets.CODECOV_TOKEN }}
```

## Inputs and secrets

| Name | Kind | Default | Purpose |
|---|---|---:|---|
| `ruby-version` | input | `3.3.5` | Ruby used to test and build the gem. |
| `CODECOV_TOKEN` | optional secret | none | Upload `coverage/coverage.xml` to Codecov. |

When no Codecov token is mapped, coverage is still validated and retained as a
GitHub Actions artifact.

## Execution

The workflow performs these steps in one job:

1. Check out the caller repository and triggering commit.
2. Install Ruby and cache the bundle.
3. Run `COVERAGE=true bundle exec rake test`.
4. Verify `coverage/coverage.xml`.
5. Run `bundle exec rake build`.
6. Upload the coverage directory and generated gem.
7. Upload coverage to Codecov when configured.

Any failed command stops the job. Missing coverage or gem files are treated as
errors.

## Testing multiple Ruby versions

The main workflow represents one Ruby version per call. If a gem needs a Ruby
compatibility matrix, create one caller job for each supported version so every
result and artifact has an explicit name:

```yaml
jobs:
  ruby-3-2:
    uses: LesliTech/LesliActions/.github/workflows/lesli-ci-gem.yaml@master
    with:
      ruby-version: "3.2"
    secrets:
      CODECOV_TOKEN: ${{ secrets.CODECOV_TOKEN }}

  ruby-3-3:
    uses: LesliTech/LesliActions/.github/workflows/lesli-ci-gem.yaml@master
    with:
      ruby-version: "3.3"
    secrets:
      CODECOV_TOKEN: ${{ secrets.CODECOV_TOKEN }}
```

Use the oldest supported Ruby version as part of the required checks when the
gemspec advertises compatibility with it.
