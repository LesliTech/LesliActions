<div align="center" class="documentation-header">
    <img width="100" alt="LesliActions logo" src="./actions-logo.svg" />
    <h3 align="center">Reusable GitHub Actions workflows for the Lesli Framework.</h3>
</div>

<br />

## Overview

LesliActions provides readable and reusable continuous integration workflows for
Rails host applications, Rails engines, Rails gems, and standalone Ruby gems.

Each consuming repository owns its triggers. LesliActions owns the test stages,
tool configuration, coverage artifacts, and integration environments.

## Workflows

```text
.github/workflows/
├── lesli-ci-app.yaml
├── lesli-ci-engine.yaml
├── lesli-ci-gem.yaml
├── lesli-test-integration.yaml
├── lesli-test-integration-new.yaml
└── lesli-test-security.yaml
```

| Workflow | Purpose |
|---|---|
| `lesli-ci-app.yaml` | Test a Rails application that hosts Lesli modules. |
| `lesli-ci-engine.yaml` | Test a Lesli engine or a gem that implements `Rails::Engine`. |
| `lesli-ci-gem.yaml` | Test and build a standalone Ruby gem. |
| `lesli-test-integration.yaml` | Test the caller module inside the real LesliBuilder host. |
| `lesli-test-integration-new.yaml` | Install the caller module inside a newly generated Rails application. |
| `lesli-test-security.yaml` | Run Brakeman and preserve its machine-readable report. |

The three `lesli-ci-*` workflows are the main entry points. The
`lesli-test-*` workflows are reusable stages composed by the CI workflows.

## Execution order

Engine stages run in this order:

```text
Security
└── Engine tests and gem build
    └── LesliBuilder integration
        └── New Rails application integration
```

A failed stage prevents every dependent stage from starting. This keeps failure
feedback focused and avoids consuming runners after the result is already known.

## Rails application

Use `lesli-ci-app.yaml` for a host application such as LesliBuilder:

```yaml
name: Lesli CI

on:
  push:
    branches: [master]
  pull_request:
  workflow_dispatch:

jobs:
  ci:
    uses: LesliTech/LesliActions/.github/workflows/lesli-ci-app.yaml@master
    with:
      ruby-version: "3.3.5"
      run-security: true
      run-system-tests: true
    secrets:
      CODECOV_TOKEN: ${{ secrets.CODECOV_TOKEN }}
```

## Rails engine or Rails gem

Use `lesli-ci-engine.yaml` for traditional Lesli engines and gems that define a
`Rails::Engine`.

An engine stored under `engines/` can use the defaults:

```yaml
jobs:
  ci:
    uses: LesliTech/LesliActions/.github/workflows/lesli-ci-engine.yaml@master
    with:
      module-directory: "engines"
      run-security: true
      run-integration: true
      run-integration-new: false
    secrets:
      CODECOV_TOKEN: ${{ secrets.CODECOV_TOKEN }}
```

A Rails gem stored under `gems/` uses the same workflow with a different host
directory. Security can be disabled when the gem does not contain a Brakeman
compatible Rails application:

```yaml
jobs:
  ci:
    uses: LesliTech/LesliActions/.github/workflows/lesli-ci-engine.yaml@master
    with:
      module-directory: "gems"
      run-security: false
      run-integration: false
      run-integration-new: false
    secrets:
      CODECOV_TOKEN: ${{ secrets.CODECOV_TOKEN }}
```

Every engine and Rails gem must expose these Rake tasks:

```shell
bundle exec rake test
bundle exec rake build
```

## Standalone Ruby gem

Use `lesli-ci-gem.yaml` for gems such as Termline and LesliTesting:

```yaml
jobs:
  ci:
    uses: LesliTech/LesliActions/.github/workflows/lesli-ci-gem.yaml@master
    with:
      ruby-version: "3.3.5"
    secrets:
      CODECOV_TOKEN: ${{ secrets.CODECOV_TOKEN }}
```

Standalone gems must expose the same `test` and `build` Rake tasks. Tests must
generate `coverage/coverage.xml` when `COVERAGE=true`.

## Inputs

All workflows use Ruby `3.3.5` by default. Engine integration can additionally
configure:

- `module-directory`: `engines` or `gems`.
- `run-security`: run Brakeman before tests.
- `run-integration`: test inside LesliBuilder.
- `run-integration-new`: test inside a newly generated Rails application.
- `builder-ref`: LesliBuilder branch, tag, or commit.
- `lesli-ref`: Lesli branch, tag, or commit.
- `actions-ref`: LesliActions branch, tag, or commit used for host configuration.
- `rails-version`: Rails version used to generate the new application.

Input descriptions and defaults are also declared directly in every workflow.

## Secrets

`CODECOV_TOKEN` is optional. Coverage remains available as a workflow artifact
when the token is not provided.

## Historical workflows

The first generation workflows are preserved under
`.github/workflows/historical/`. GitHub only loads files placed directly in
`.github/workflows`, so historical files are intentionally inactive.

## Versioning

Examples currently reference `@master` while the new contracts are being
introduced. Once they are verified across the Lesli repositories, publish a
`v1` tag and update callers to use `@v1`.

## Documentation

- [Lesli website](https://www.lesli.dev/)
- [Lesli documentation](https://www.lesli.dev/)
- [Source code](https://github.com/LesliTech/LesliActions)

## License

Copyright (c) 2026, Lesli Technologies, S. A.

LesliActions is distributed under the GNU General Public License version 3 or
later. See the `license` file for details.
