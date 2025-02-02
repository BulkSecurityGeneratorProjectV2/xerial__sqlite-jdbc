# Development
## Prerequisites

- JDK 8
- JDK 11 is required to run the `spotless` plugin for code formatting

## Commits

Since August 2022 the commit messages follow the [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/) standard. This allows for easier and better release notes generation.

## How to submit a patch

1. Fork this project on GitHub
2. make some changes
3. `git commit -m 'type: what changes are made to the source'`
4. `git push`
5. Create a pull request

:warning: Do not force push to your PR branch. This makes the reviewer's job more difficult because it clears what changes have already been reviewed or not. The commits will be squashed while merging anyway.

# How to compile the native libraries
## Prerequisites

1. JDK 8
2. Perl
3. Maven
4. make
5. gcc
6. curl
7. unzip
8. Docker (for cross-compilation only)

## Version update
1. Edit the `VERSION` file and set the SQLite version to use.
2. Edit the version number in `pom.xml` to `${VERSION}.0-SNAPSHOT`. So if `VERSION`=`3.39.2`, the version number in `pom.xml` should be `3.39.2.0-SNAPSHOT`. 

## Build for the current platform
```shell
# For the current platform
$ make native

# For Apple Silicon (M1). Need to install a jdk compiled for M1 https://github.com/microsoft/openjdk-aarch64
$ JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk-16+10/Contents/Home make native
```

## Build for all platforms
The native library is cross-compiled for different OS and architecture by using Docker.
```shell
$ make native-all
```

On Windows it is recommended to use WSL2.

You can check the `native-all` goal in `Makefile` for a list of available targets.

# Release process
The project version can change by 2 means:
1. By changing the bundled version of SQLite, in which case the project version changes to align with the SQLite version. This is a manual process for now.
2. When triggering a release. This is done automatically through GitHub Actions.

## Trigger a release
A release can be triggered from GitHub Actions by [manually running](https://docs.github.com/en/actions/managing-workflow-runs/manually-running-a-workflow) the CI workflow.

## What happens when performing a release?

Multiple actions will happen in sequence, all orchestrated by the GitHub workflow:
1. The version in `pom.xml` is changed to remove the `-SNAPSHOT`. This is done by using the [Maven Versions plugin](https://www.mojohaus.org/versions-maven-plugin/).
2. Deploy to Maven Central. This is done by using the Maven `deploy` goal with the `release` profile. It relies on credentials stored in the repository's secrets.
3. Perform a release commit with the changed `pom.xml`, as well as creating a git tag with the version number.
4. Create a GitHub release. This is done via JReleaser, and will also include a changelog since the last release.
5. The version in `pom.xml` is incremented for the next snapshot.
6. Perform a commit with the version updated for the next snapshot.

## Snapshot publishing

The CI workflow will also publish a new snapshot to [Sonatype's snapshots repository](https://oss.sonatype.org/content/repositories/snapshots/org/xerial/sqlite-jdbc/) whenever a change occurs.