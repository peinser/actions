# Install Proton Pass CLI Action

This composite action installs the Proton Pass CLI (`pass-cli`) on the current GitHub Actions runner.

## Inputs

- `force-install` (optional, default: `false`): installs `pass-cli` even when it is already present in `PATH`.

## Outputs

- `pass-cli-version`: the installed `pass-cli` version string.

## Usage

```yaml
- name: Install pass-cli
  uses: peinser/actions/proton-pass-install@main

- name: Check version
  run: pass-cli --version
```

## Notes

- The action uses Proton's official installer script: `https://proton.me/download/pass-cli/install.sh`.
- Use this action before `proton-pass-secret` when a workflow needs to resolve Proton Pass secrets.
