# Security scans

`lesli-test-security.yaml` provides the shared Brakeman stage for Rails
applications, Rails engines, and Rails gems with a scannable Rails application.
Standalone Ruby gems do not use this workflow.

## Requirements

The caller must include Brakeman in its bundle. A typical development dependency
is:

```ruby
group :development, :test do
  gem "brakeman", require: false
end
```

The selected `rails-path` must contain the Rails application to scan.

## Direct usage

The app and engine CI workflows call this stage automatically when security is
enabled. It can also be called directly:

```yaml
name: "Lesli Security"

on:
  pull_request:
  workflow_dispatch:

jobs:
  security:
    uses: LesliTech/LesliActions/.github/workflows/lesli-test-security.yaml@master
    with:
      ruby-version: "3.3.5"
      rails-path: "test/dummy"
```

## Inputs

| Input | Default | Purpose |
|---|---:|---|
| `ruby-version` | `3.3.5` | Ruby used to install dependencies and run Brakeman. |
| `rails-path` | `.` | Rails application directory scanned by Brakeman. |

Common values are:

- `.` for a Rails host application.
- `test/dummy` for an engine whose test suite includes a dummy application.

## Failure policy

The first Brakeman command uses its normal exit behavior. Findings or execution
errors that make Brakeman return a failure status stop the job immediately.

Only after that scan passes does the workflow generate a Sonar-compatible JSON
report with `--no-exit-on-warn`. This second invocation exists to serialize the
already accepted scan result; it does not turn a failed security gate into a
successful run.

## Report

The workflow uploads:

```text
reports/brakeman/brakeman-sonar.json
```

as the artifact:

```text
brakeman-<repository>
```

The report format can be consumed by SonarCloud or downloaded for manual
inspection. LesliActions preserves the report but does not upload it to a
specific SonarCloud project, because project keys, organization settings, and
tokens belong to the consuming repository.
