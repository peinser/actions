# Common Actions

[![Tag](https://img.shields.io/github/v/tag/peinser/actions)](https://github.com/peinser/actions/tags)
[![License](https://img.shields.io/github/license/peinser/actions)](./LICENSE)

This repository contains shared GitHub Actions used throughout the organisation.

## Available actions

- [semver](./semver): Bumps patch/minor/major versions and returns current and next versions, compliant with [SemVer 2.x](./semver/semver).
- [proton-pass-install](./proton-pass-install): Installs Proton Pass CLI (`pass-cli`) on the runner.
- [proton-pass-secret](./proton-pass-secret): Logs in with a Proton Pass PAT and resolves one `pass://` secret reference to workflow output.

## Versioning policy

Use immutable SemVer tags for releases and moving major tags for consumption:

- Release tags: `vMAJOR.MINOR.PATCH` (for example `v1.2.0`).
- Major tags: `vMAJOR` (for example `v1`) updated to the latest compatible release.

Consumers should reference major tags (for example `@v1`) instead of `@main`.

See [VERSIONING.md](./VERSIONING.md) for the release flow.
