# Integration tests

Integration workflows verify that an engine or Rails gem works outside its own
repository. They check out the exact caller commit, install it from a local path,
prepare a host, and run the host test suite.

The main `lesli-ci-engine.yaml` workflow composes both integration strategies.
They can also be called directly when a repository needs a specialized CI
layout.

## Strategies

| Workflow | Host | Database | Best for |
|---|---|---|---|
| `lesli-test-integration.yaml` | Existing LesliBuilder checkout | SQLite configuration from LesliActions | Compatibility with the real development host |
| `lesli-test-integration-new.yaml` | Newly generated Rails application | PostgreSQL 16 | Missing dependencies, installation assumptions, and clean-host compatibility |

These strategies are complementary. Passing inside LesliBuilder proves
ecosystem compatibility; passing inside a new application proves the module can
be assembled without undeclared help from the development monorepo.

## Real LesliBuilder host

The real-host workflow:

1. Validates that `module-directory` is `engines` or `gems`.
2. Checks out the selected LesliBuilder ref at the workspace root.
3. Checks out the exact caller commit into the selected module directory.
4. Checks out Lesli core unless Lesli itself is under test.
5. Loads the shared Gemfile and SQLite database configuration from
   LesliActions.
6. Adds the caller to `Gemfile.lesli` when it is not already declared.
7. Installs dependencies and rebuilds the test database.
8. Runs the complete host suite with `LESLI_INTEGRATION_TEST=true`.

Direct usage:

```yaml
jobs:
  integration:
    uses: LesliTech/LesliActions/.github/workflows/lesli-test-integration.yaml@master
    with:
      ruby-version: "3.3.5"
      module-directory: "engines"
      builder-ref: "master"
      lesli-ref: "master"
      actions-ref: "master"
```

### Inputs

| Input | Default | Purpose |
|---|---:|---|
| `ruby-version` | `3.3.5` | Ruby used by the host. |
| `module-directory` | `engines` | Caller destination: `engines` or `gems`. |
| `builder-ref` | `master` | LesliBuilder host branch, tag, or commit. |
| `lesli-ref` | `master` | Lesli core branch, tag, or commit. |
| `actions-ref` | `master` | LesliActions configuration branch, tag, or commit. |

## Newly generated Rails host

The new-host workflow:

1. Starts PostgreSQL 16.
2. Validates `module-directory`.
3. Installs the requested Ruby and exact Rails version.
4. Generates `LesliApp` without installing dependencies yet.
5. Checks out the exact caller commit, Lesli core, and LesliActions
   configuration inside the application.
6. Adds `Gemfile.lesli` and the PostgreSQL test configuration.
7. Installs dependencies once every local path is ready.
8. Rebuilds the database and eager loads the application.
9. Runs the generated application's tests with
   `LESLI_INTEGRATION_TEST=true`.

Direct usage:

```yaml
jobs:
  integration-new:
    uses: LesliTech/LesliActions/.github/workflows/lesli-test-integration-new.yaml@master
    with:
      ruby-version: "3.3.5"
      rails-version: "8.1.4"
      module-directory: "engines"
      lesli-ref: "master"
      actions-ref: "master"
```

### Inputs

| Input | Default | Purpose |
|---|---:|---|
| `ruby-version` | `3.3.5` | Ruby used to generate and test the host. |
| `rails-version` | `8.1.4` | Exact Rails version installed for generation. |
| `module-directory` | `engines` | Caller destination: `engines` or `gems`. |
| `lesli-ref` | `master` | Lesli core branch, tag, or commit. |
| `actions-ref` | `master` | LesliActions configuration branch, tag, or commit. |

## Ref selection

Integration refs are independent:

- `builder-ref` selects the real host baseline.
- `lesli-ref` selects Lesli core.
- `actions-ref` selects shared Gemfile and database configuration.

Use commits or release tags when reproducing a historical failure. Use branches
when validating coordinated changes across repositories.

There is no `branch` input for the module under test. GitHub's checkout action
uses the caller repository and the exact commit associated with the workflow
run, so pull requests are tested without guessing a branch name.
