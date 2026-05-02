# Get Proton Pass Secret Action

This composite action logs in to Proton Pass with a Personal Access Token (PAT) and resolves one secret into a GitHub Actions output.

## What this action does

1. Uses `PROTON_PASS_PERSONAL_ACCESS_TOKEN` to authenticate via `pass-cli login`.
2. Resolves either:
   - a provided `pass://...` secret reference, or
   - a generated reference from `vault_share_id` + (`item_id` or `item_title`) + `field`.
3. Masks the resolved value in workflow logs.
4. Exposes the resolved value as an output.
5. Optionally logs out (`logout: true` by default).

## Inputs

- `personal-access-token` (required): Proton Pass PAT string.
- `secret-reference` (optional): secret reference in `pass://vault/item/field` format.
- `vault_share_id` (optional): vault Share ID when you do not pass `secret-reference`.
- `item_id` (optional): item ID used with `vault_share_id`.
- `item_title` (optional): item title used with `vault_share_id`.
- `field` (optional, default: `note`): field name used with `vault_share_id` + item selector.
- `output-name` (optional, default: `secret`): output key for the resolved secret.
- `logout` (optional, default: `true`): whether to run `pass-cli logout` when done.

Input rules:

- Either `secret-reference` OR `vault_share_id` must be provided.
- When using `vault_share_id`, provide exactly one of `item_id` or `item_title`.

## Outputs

- `secret`: resolved secret value.
- `<output-name>`: same resolved secret value, but published under your chosen output name.

## Requirements and assumptions

- The workflow runner can reach Proton services over the network.
- `pass-cli` is installed and available in `PATH` before this action runs.
  - Recommended: use `peinser/actions/proton-pass-install` first.
- The PAT grants access to the target vault/item/field.
- The provided secret reference is valid (`pass://vault/item/field`), or `vault_share_id` + item selector resolves to an existing item.
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
        uses: peinser/actions/proton-pass-install@v1

      - name: Resolve API token
        id: proton_secret
        uses: peinser/actions/proton-pass-secret@v1
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

Using `vault_share_id` + `item_id`:

```yaml
- name: Resolve secret by share and item ID
  id: proton_secret
  uses: peinser/actions/proton-pass-secret@v1
  with:
    personal-access-token: ${{ secrets.PROTON_PASS_PAT }}
    output-name: kubeconfig
    vault_share_id: kxMSS9Ok_rQjUdXujFCR2w5w43oEtH9bGpq5gvMry_buisSpwbzfdiWdIrJJ7_fbf8a1xFjFzXcku1VmTPSfIQ==
    item_id: meepmeep
    field: note
```

Using `vault_share_id` + `item_title`:

```yaml
- name: Resolve secret by share and item title
  id: proton_secret
  uses: peinser/actions/proton-pass-secret@v1
  with:
    personal-access-token: ${{ secrets.PROTON_PASS_PAT }}
    output-name: kubeconfig
    vault_share_id: kxMSS9Ok_rQjUdXujFCR2w5w43oEtH9bGpq5gvMry_buisSpwbzfdiWdIrJJ7_fbf8a1xFjFzXcku1VmTPSfIQ==
    item_title: KUBECONFIG
    field: note
```

## Security notes

- Prefer PAT login for CI; avoid username/password automation where possible.
- Keep PAT scope narrow (least privilege).
- Never print secret outputs directly to logs.
- Use GitHub encrypted secrets for PAT storage.
