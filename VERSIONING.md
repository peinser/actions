# Action Versioning

This repository follows GitHub Actions versioning best practices.

## Tag strategy

- Create immutable release tags in SemVer format: `vMAJOR.MINOR.PATCH`.
- Maintain a moving major tag (`vMAJOR`) for each major line.
- Workflows consuming actions from this repo should use `@vMAJOR`.

Examples:

- `uses: peinser/actions/semver@v1`
- `uses: peinser/actions/proton-pass-install@v1`
- `uses: peinser/actions/proton-pass-secret@v1`

## Why this approach

- `@main` is mutable and can break consumers unexpectedly.
- Patch/minor updates can flow to users of `@v1` while preserving compatibility.
- Full release tags (`v1.2.3`) remain reproducible for audits and rollbacks.

## Release process

1. Merge changes into `main`.
2. Create a release tag `vMAJOR.MINOR.PATCH` from the target commit.
3. Push the release tag.
4. The `update-major-action-tag` workflow updates the corresponding `vMAJOR` tag.

Example:

```bash
git tag v1.0.0
git push origin v1.0.0
```

After pushing `v1.0.0`, the workflow updates `v1` to the same commit.

## Initial setup required

No tags exist yet in this repository. Create and push the first release tag (for example `v1.0.0`) to initialize `v1`.
