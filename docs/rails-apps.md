# Rails applications

Use `lesli-ci-app.yaml` for a Rails application that hosts Lesli modules. The
workflow can run a security scan, prepare the test database, run regular tests,
optionally run system tests, and publish coverage.

## Requirements

The application must provide:

- A `Gemfile` and a committed `Gemfile.lock` compatible with Linux.
- An executable `bin/rails`.
- A test database configuration that works on the GitHub Ubuntu runner.
- LesliTesting or equivalent coverage configuration that writes
  `coverage/coverage.xml` when `COVERAGE=true`.
- Brakeman in the bundle when `run-security` is enabled.

The workflow runs `bin/rails db:test:prepare`; any required service database
must be configured by the application or by a future application-specific
workflow extension.

## Caller workflow

Create `.github/workflows/lesli-ci.yaml` in the application repository:

```yaml
name: "Lesli CI"

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
      run-system-tests: false
    secrets:
      CODECOV_TOKEN: ${{ secrets.CODECOV_TOKEN }}
```

The consuming repository owns the `push`, `pull_request`, and manual triggers.
Remove or change them to match that repository's release policy.

## Inputs

| Input | Type | Default | Purpose |
|---|---|---:|---|
| `ruby-version` | string | `3.3.5` | Ruby used to install and test the application. |
| `run-security` | boolean | `true` | Run Brakeman before application tests. |
| `run-system-tests` | boolean | `false` | Run `bin/rails test:system` after regular tests pass. |

## Execution

When every optional feature is enabled, the workflow runs:

1. Brakeman against the repository root.
2. `bin/rails db:test:prepare`.
3. `COVERAGE=true bin/rails test`.
4. `COVERAGE=true bin/rails test:system`.
5. Verification of `coverage/coverage.xml`.
6. Coverage artifact upload.
7. Optional Codecov upload.

A failed security scan prevents the test job from starting. A failed test step
prevents the remaining test steps from running.

## Coverage and Codecov

`CODECOV_TOKEN` is optional. Configure it as a GitHub Actions repository secret
and map it as shown above to upload coverage to Codecov.

When the secret is unavailable, including many workflows triggered from forks,
the Codecov step is skipped. The `coverage-<repository>-<ruby>` artifact is still
required and uploaded.

## System tests

System tests are disabled by default because they may require browser packages,
drivers, or application-specific services. Enable them only after the host can
run `bin/rails test:system` on an Ubuntu runner without interactive setup.

If regular and system tests write coverage in separate processes, configure
SimpleCov result merging so the final XML contains both suites.
