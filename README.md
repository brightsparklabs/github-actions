# Github Actions

Central repo for re-using actions across all other projects.

# Reusable Workflows

This repo contains the following reusable workflows that can be accessed from other repos.

The `<version>` referenced below can be a SHA, a release tag, or a branch name. Using a release tag
is preferred to avoid accidentally running breaking changes.

## Build with Gradle

A reusable Gradle build workflow for testing for breaking changes. Runs `./gradlew build` using JDK
17.

### Inputs

| Name              | Description                                                              | Required | Type     | Default         |
|-------------------|--------------------------------------------------------------------------|----------|----------|-----------------|
| `github-ref-name` | The short ref name of the branch or tag that triggered the workflow run. | `true`   | `string` |                 |
| `os`              | The operating system to run the Gradle build on.                         | `false`  | `string` | `ubuntu-latest` |

### Secrets

No secrets need to be passed to this workflow.

### Usage

By default, `./gradlew build` will run on `ubuntu-latest`.

```yaml
# exmaple-repo/.github/workflows/test.yml

name: Test
on:
  # Call workflow on push.
  push:
  # Call workflow when called from other workflows.
  workflow_call:

jobs:
  call-test-gradle-build-workflow:
    uses: brightsparklabs/github-actions/.github/workflows/test-gradle-build.yml@<version>
    with:
      github-ref-name: ${GITHUB_REF_NAME}
```

 Multiple operating systems can be tested using a [matrix](https://docs.github.com/en/actions/using-jobs/using-a-matrix-for-your-jobs):

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
      github-ref-name: ${GITHUB_REF_NAME}
      os: ${{ matrix.os }}
```

## Dependabot auto-merge

A reusable Dependabot auto-merge workflow for merging patch and minor updates. Pull requests will
still be created for major updates, however they will need to be merged manually.

### Inputs

| Name     | Description                                      | Required | Type     |
|----------|--------------------------------------------------|----------|----------|
| `pr-url` | The URL of the Dependabot PR to auto-merge.      | `true`   | `string` |

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
    # See https://docs.github.com/en/actions/using-jobs/using-jobs-in-a-workflow#defining-prerequisite-jobs
    needs: call-test-workflow
    uses: brightsparklabs/github-actions/.github/workflows/dependabot-auto-merge.yml@<version>
    with:
      pr-url: ${{ github.event.pull_request.html_url }}
    # Since all of our workflows are called from the same organisation, we can use the `inherit`
    # keyword to pass secrets to the called workflow.
    # See https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_idsecretsinherit
    secrets: inherit
```

## Publish to Maven Central

A reusable workflow for publishing packages to Maven Central. Runs
`./gradlew -i publishToMavenCentral` using JDK 17.

### Inputs

| Name              | Description                                                              | Required | Type     | Default         |
|-------------------|--------------------------------------------------------------------------|----------|----------|-----------------|
| `github-ref-name` | The short ref name of the branch or tag that triggered the workflow run. | `true`   | `string` |                 |

### Secrets

The following secrets must be passed to this workflow:
- `PGP_SIGNING_KEY`
- `PGP_SIGNING_PASSWORD`
- `MAVEN_CENTRAL_USERNAME`
- `MAVEN_CENTRAL_PASSWORD`

### Usage

```yaml
# exmaple-repo/.github/workflows/publish.yml

name: Publish to Maven Central
on:
  # Call workflow on push to the `master` branch.
  push:
    branches:
      - master

jobs:
  call-test-workflow:
    # Run the locally defined reusable test workflow.
    uses: ./.github/workflows/test.yml
  call-publish-to-maven-central-workflow:
    # Specify that this job should only run if the test job succeeds.
    # See https://docs.github.com/en/actions/using-jobs/using-jobs-in-a-workflow#defining-prerequisite-jobs
    needs: call-test-workflow
    uses: brightsparklabs/github-actions/.github/workflows/publish-to-maven-central.yml@<version>
    with:
      github-ref-name: ${GITHUB_REF_NAME}
    # Since all of our workflows are called from the same organisation, we can use the `inherit`
    # keyword to pass secrets to the called workflow.
    # See https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_idsecretsinherit
    secrets: inherit
```
