# Rails engines and Rails gems

Use `lesli-ci-engine.yaml` for both traditional Lesli engines and gems that
implement `Rails::Engine`. They share the same test and build contract; the
`module-directory` input tells integration hosts where to install the caller.

## Requirements

The repository must provide:

```shell
bundle exec rake test
bundle exec rake build
```

The test task must generate `coverage/coverage.xml` when `COVERAGE=true`, and the
build task must generate a package under `pkg/*.gem`.

When security is enabled, Brakeman must be present in the bundle and
`security-path` must point to a Rails application Brakeman can scan. An engine
with a dummy application commonly uses `test/dummy`.

## Engine caller

An engine installed under `engines/` can use:

```yaml
name: "Lesli CI"

on:
  push:
    branches: [master]
  pull_request:
  workflow_dispatch:

jobs:
  ci:
    uses: LesliTech/LesliActions/.github/workflows/lesli-ci-engine.yaml@master
    with:
      ruby-version: "3.3.5"
      module-directory: "engines"
      security-path: "test/dummy"
      run-security: true
      run-integration: true
      run-integration-new: false
    secrets:
      CODECOV_TOKEN: ${{ secrets.CODECOV_TOKEN }}
```

Use `security-path: "."` instead when the repository root itself is a complete
Rails application.

## Rails gem caller

A Rails gem installed under `gems/` uses the same main workflow:

```yaml
name: "Lesli CI"

on:
  push:
    branches: [master]
  pull_request:
  workflow_dispatch:

jobs:
  ci:
    uses: LesliTech/LesliActions/.github/workflows/lesli-ci-engine.yaml@master
    with:
      ruby-version: "3.3.5"
      module-directory: "gems"
      run-security: false
      run-integration: true
      run-integration-new: false
    secrets:
      CODECOV_TOKEN: ${{ secrets.CODECOV_TOKEN }}
```

Disabling security is appropriate when the gem does not contain a Rails
application that Brakeman can inspect. It does not weaken or skip the gem's test,
build, or requested integration stages.

## Inputs

| Input | Type | Default | Purpose |
|---|---|---:|---|
| `ruby-version` | string | `3.3.5` | Ruby used by every stage. |
| `rails-version` | string | `8.1.4` | Rails used by the generated-host integration. |
| `module-directory` | string | `engines` | Install the caller under `engines` or `gems`. |
| `security-path` | string | `.` | Rails application path scanned by Brakeman. |
| `run-security` | boolean | `true` | Run Brakeman before unit tests. |
| `run-integration` | boolean | `true` | Test inside the real LesliBuilder host. |
| `run-integration-new` | boolean | `false` | Test inside a newly generated Rails application. |
| `builder-ref` | string | `master` | LesliBuilder ref used by host integration. |
| `lesli-ref` | string | `master` | Lesli core ref used by integration. |
| `actions-ref` | string | `master` | LesliActions ref supplying host configuration. |

`module-directory` accepts only `engines` or `gems`. Integration stops with a
clear error for any other value.

## Execution order

```text
Security
└── Tests, coverage, and gem build
    └── LesliBuilder integration
        └── Generated Rails application integration
```

The generated-host stage depends on the test job and, when enabled, the
LesliBuilder integration. It can still run when `run-integration` is deliberately
disabled, but it never runs after a failure or cancellation.

## Recommended adoption

Start with unit tests and the real host integration:

```yaml
run-security: true
run-integration: true
run-integration-new: false
```

Enable `run-integration-new` once the module declares every dependency it needs
and can install into a clean application. This stage is intentionally stricter
than testing inside LesliBuilder, where other modules may provide dependencies
implicitly.
