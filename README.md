# Github Actions

Central repo for re-using actions across all other projects.

# Reusable Workflows

This repo contains the following reusable workflows that can be accessed from other repos.

The `<version>` referenced below can be a SHA, a release tag, or a branch name. Using a release tag
is preferred to avoid accidentally running breaking changes.

## Java Workflow

A reusable standardised Java workflow to automate testing, dependency patching, and release
publishing. This workflow calls the following workflows within this repo:
- Build with Gradle
- Dependabot Auto-Merge

In addition, it defines and runs a job for publishing the project as a package to Maven Central; this is only done when pushing to master, after first getting the correct tag/version for the project in question.

### Inputs

| Name | Description                                                                    | Required | Type     | Default               |
|------|--------------------------------------------------------------------------------|----------|----------|-----------------------|
| `os` | A JSON string containing the list of operating systems to run Gradle build on. | `false`  | `string` | `'["ubuntu-latest"]'` |

### Secrets

The following secrets must be passed to this workflow:
- `GITHUB_TOKEN`

### Usage

**NOTE: Its easiest to use `@master` for the version unless you really need to be explicit.**

```yaml
name: Java Workflow
on: [push, pull_request]

jobs:
  call-java-workflow:
    uses: brightsparklabs/github-actions/.github/workflows/java.yml@<version>
    # Since all of our workflows are called from the same organisation, we can use the `inherit`
    # keyword to pass secrets to the called workflow.
    secrets: inherit
    # These permissions are required for Dependabot to merge PRs.
    permissions:
      contents: write
      pull-requests: write
```

By default, `./gradlew build` will run on `ubuntu-latest`. Multiple operating systems can be
specified with the `os` input:

```yaml
jobs:
  call-java-workflow:
    uses: brightsparklabs/github-actions/.github/workflows/java.yml@<version>
    secrets: inherit
    permissions:
      contents: write
      pull-requests: write
    with:
      os: '["ubuntu-latest", "windows-latest"]'
```

## Build with Gradle

A reusable Gradle build workflow for testing for breaking changes. Runs `./gradlew build` using JDK
17.

### Inputs

| Name | Description                                      | Required | Type     | Default         |
|------|--------------------------------------------------|----------|----------|-----------------|
| `os` | The operating system to run the Gradle build on. | `false`  | `string` | `ubuntu-latest` |

### Secrets

No secrets need to be passed to this workflow.

### Usage

```yaml
# exmaple-repo/.github/workflows/test.yml

name: Test
on: [push, workflow_call]

jobs:
  call-test-gradle-build-workflow:
    uses: brightsparklabs/github-actions/.github/workflows/test-gradle-build.yml@<version>
```

By default, `./gradlew build` will run on `ubuntu-latest`. Multiple operating systems can be tested
using a [matrix](https://docs.github.com/en/actions/using-jobs/using-a-matrix-for-your-jobs):

```yaml
jobs:
  call-test-gradle-build-workflow:
    strategy:
      matrix:
        # Test on the following operating systems.
        os:
          - ubuntu-latest
          - windows-latest
    uses: brightsparklabs/github-actions/.github/workflows/test-gradle-build.yml@<version>
    with:
      os: ${{ matrix.os }}
```

## Dependabot Auto-Merge

A reusable Dependabot auto-merge workflow for merging patch and minor updates. Pull requests will
still be created for major updates, however they will need to be merged manually.

### Inputs

This workflow has no inputs.

### Secrets

The following secrets must be passed to this workflow:
- `GITHUB_TOKEN`

### Usage

```yaml
# exmaple-repo/.github/workflows/dependabot-test-and-auto-merge.yml

name: Dependabot test and auto-merge
on: pull_request

permissions:
  contents: write
  pull-requests: write

jobs:
  call-test-workflow:
    # Run the locally defined reusable test workflow.
    uses: ./.github/workflows/test.yml
  call-dependabot-auto-merge-workflow:
    # Specify that this job should only run if the test job succeeds.
    needs: call-test-workflow
    uses: brightsparklabs/github-actions/.github/workflows/dependabot-auto-merge.yml@<version>
    secrets: inherit
```

## Gradle Plugins Workflow

A reusable standardised Gradle workflow to automate testing, dependency patching, and release publishing. This workflow calls the following workflows within this repo:
- Build with Gradle
- Dependabot Auto-Merge

In addition, it defines and runs a job for publishing gradle plugins when pushing to master, after first getting the correct tag/version for the project in question.

### Inputs

| Name | Description                                                                    | Required | Type     | Default               |
|------|--------------------------------------------------------------------------------|----------|----------|-----------------------|
| `os` | A JSON string containing the list of operating systems to run Gradle build on. | `false`  | `string` | `'["ubuntu-latest"]'` |

### Secrets

The following secrets must be passed to this workflow:
- `GITHUB_TOKEN`

### Usage

**NOTE: Its easiest to use `@master` for the version unless you really need to be explicit.**

```yaml
name: Gradle Plugins Workflow
on: [push, pull_request]

jobs:
  call-gradle-plugins-workflow:
    uses: brightsparklabs/github-actions/.github/workflows/gradle-plugins.yml@<version>
    secrets: inherit
    # These permissions are required for Dependabot to merge PRs.
    permissions:
      contents: write
      pull-requests: write
```

By default, `./gradlew build` will run on `ubuntu-latest`. Multiple operating systems can be
specified with the `os` input:

```yaml
jobs:
  call-gradle-plugins-workflow:
    uses: brightsparklabs/github-actions/.github/workflows/gradle-plugins.yml@<version>
    secrets: inherit
    permissions:
      contents: write
      pull-requests: write
    with:
      os: '["ubuntu-latest", "windows-latest"]'
```
