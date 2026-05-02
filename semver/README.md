# SemVer Action

This action provides semantic version operations using the bundled `semver` script.

## Inputs

- `current-version` (required): current application version.
- `bump` (required): one of `patch`, `minor`, or `major`.

## Outputs

- `current`: release version.
- `next`: next development version.
