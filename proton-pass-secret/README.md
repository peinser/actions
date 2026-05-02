# Get Proton Pass Secret Action

This composite action logs in to Proton Pass with a Personal Access Token (PAT) and resolves one secret reference (`pass://vault/item/field`) into a GitHub Actions output.

## What this action does

1. Uses `PROTON_PASS_PERSONAL_ACCESS_TOKEN` to authenticate via `pass-cli login`.
2. Resolves the provided `pass://...` secret reference using `pass-cli run`.
3. Masks the resolved value in workflow logs.
4. Exposes the resolved value as an output.
5. Optionally logs out (`logout: true` by default).

## Inputs

- `personal-access-token` (required): Proton Pass PAT string.
- `secret-reference` (required): secret reference in `pass://vault/item/field` format.
- `output-name` (optional, default: `secret`): output key for the resolved secret.
- `logout` (optional, default: `true`): whether to run `pass-cli logout` when done.

## Outputs

- `secret`: resolved secret value.
- `<output-name>`: same resolved secret value, but published under your chosen output name.

## Requirements and assumptions

- The workflow runner can reach Proton services over the network.
- `pass-cli` is installed and available in `PATH` before this action runs.
  - Recommended: use `peinser/actions/proton-pass-install` first.
- The PAT grants access to the target vault/item/field.
- The secret reference is valid and complete (`pass://vault/item/field`).
- This action currently treats an empty resolved value as an error.

## How to create and configure a PAT for CI

Use Proton Pass CLI PAT capabilities to create a dedicated token with minimal access for your pipeline.

1. Install `pass-cli` locally and log in interactively (outside CI).
2. Create a PAT with a clear name (for example: `github-actions-my-repo`).
3. Restrict PAT access to only the vaults/items needed by your workflow.
4. Copy the generated PAT value.
5. Store it as a GitHub secret (for example `PROTON_PASS_PAT`) at repository or organization level.
6. In workflows, pass it to this action via `${{ secrets.PROTON_PASS_PAT }}`.
7. Rotate or revoke the PAT when no longer needed, or on a regular schedule.

For command details, see Proton documentation:

- https://protonpass.github.io/pass-cli/commands/login/
- https://protonpass.github.io/pass-cli/

## Usage

```yaml
jobs:
  example:
    runs-on: ubuntu-latest
    steps:
      - name: Install pass-cli
        uses: peinser/actions/proton-pass-install@main

      - name: Resolve API token
        id: proton_secret
        uses: peinser/actions/proton-pass-secret@main
        with:
          personal-access-token: ${{ secrets.PROTON_PASS_PAT }}
          secret-reference: pass://CI Vault/Service API/password
          output-name: service-api-token

      - name: Use resolved secret
        env:
          SERVICE_API_TOKEN: ${{ steps.proton_secret.outputs.service-api-token }}
        run: |
          test -n "${SERVICE_API_TOKEN}"
          echo "Secret was resolved and injected into env"
```

## Security notes

- Prefer PAT login for CI; avoid username/password automation where possible.
- Keep PAT scope narrow (least privilege).
- Never print secret outputs directly to logs.
- Use GitHub encrypted secrets for PAT storage.
