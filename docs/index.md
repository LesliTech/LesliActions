# LesliActions documentation

LesliActions provides a common continuous integration interface for the Lesli
ecosystem. A consuming repository defines when CI runs; LesliActions defines how
the project is installed, tested, packaged, scanned, and integrated.

## Choose a workflow

| Project | Main workflow | Typical examples |
|---|---|---|
| Rails host application | `lesli-ci-app.yaml` | LesliBuilder and other host applications |
| Rails engine | `lesli-ci-engine.yaml` | LesliAdmin, LesliShield |
| Gem that implements `Rails::Engine` | `lesli-ci-engine.yaml` | LesliAssets, LesliView |
| Standalone Ruby gem | `lesli-ci-gem.yaml` | Termline, LesliTesting |

Use one main `lesli-ci-*` workflow from each repository. The main workflows
compose the lower-level `lesli-test-*` workflows when security or integration
checks are enabled.

## Guides

- [Rails applications](rails-apps.md)
- [Rails engines and Rails gems](rails-engines.md)
- [Standalone Ruby gems](ruby-gems.md)
- [Integration tests](integration-tests.md)
- [Security scans](security.md)
- [Migration from historical workflows](migration.md)

## Common contract

All project types use Ruby `3.3.5` by default and install dependencies through
Bundler. The caller may select another Ruby version with the `ruby-version`
input.

Projects tested by `lesli-ci-engine.yaml` or `lesli-ci-gem.yaml` must provide:

```shell
bundle exec rake test
bundle exec rake build
```

The test task runs with `COVERAGE=true` and must generate:

```text
coverage/coverage.xml
```

The build task must generate at least one package under `pkg/*.gem`. Missing
coverage or package files fail the workflow instead of silently producing an
incomplete CI result.

Rails applications are tested through `bin/rails test`. They must also generate
`coverage/coverage.xml` when coverage is enabled.

## Failure behavior

Steps use the default fail-fast behavior: when a command fails, the current job
stops. Dependent jobs are not started after a failure or cancellation.

Optional stages may be skipped deliberately. For example, disabling security
allows the test job to start, while a failed security scan prevents it from
starting.

For an engine the complete sequence is:

```text
Security
└── Engine tests and gem build
    └── LesliBuilder integration
        └── New Rails application integration
```

## Artifacts

The workflows preserve useful results directly in the GitHub Actions run:

| Artifact | Produced by | Contents |
|---|---|---|
| `coverage-<repository>-<ruby>` | App, engine, and gem CI | Complete `coverage/` directory |
| `gem-<repository>-<ruby>` | Engine and gem CI | Built packages from `pkg/*.gem` |
| `brakeman-<repository>` | Security workflow | Sonar-compatible Brakeman report |

Coverage is uploaded to Codecov only when the caller maps a non-empty
`CODECOV_TOKEN`. Artifact generation does not depend on Codecov.

## Workflow references

During adoption, examples use `@master` so repositories can consume the current
workflow definitions. After the interface is validated across the Lesli
repositories, create a stable major tag such as `v1` and update callers to
reference `@v1`.

Inputs named `builder-ref`, `lesli-ref`, and `actions-ref` control the source
versions assembled by integration tests. They do not select the caller commit:
GitHub checks out the exact commit that triggered the caller workflow.
