# Get Proton Pass Secret Action

This composite action logs in to Proton Pass with a Personal Access Token (PAT) and resolves one secret into a GitHub Actions output.

## What this action does

1. Uses `PROTON_PASS_PERSONAL_ACCESS_TOKEN` to authenticate via `pass-cli login`.
2. Resolves either:
   - a provided `pass://...` secret reference via `pass-cli run`, or
   - an item field via `pass-cli item view --share-id` + (`--item-id` or `--item-title`).
3. Base64-encodes the resolved value and exposes it as the `secret` output.
4. Masks the encoded value in workflow logs.
5. Optionally logs out (`logout: true` by default).

## Inputs

- `personal-access-token` (required): Proton Pass PAT string.
- `secret-reference` (optional): secret reference in `pass://vault/item/field` format.
- `vault_share_id` (optional): vault Share ID when you do not pass `secret-reference`.
- `item_id` (optional): item ID used with `vault_share_id`.
- `item_title` (optional): item title used with `vault_share_id`.
- `field` (optional, default: `note`): field name to extract from the item JSON. See [Supported fields](#supported-fields).
- `logout` (optional, default: `true`): whether to run `pass-cli logout` when done.

Input rules:

- Either `secret-reference` OR `vault_share_id` must be provided.
- When using `vault_share_id`, provide exactly one of `item_id` or `item_title`.

## Supported fields

The `field` input maps to a jq path on the `pass-cli item view --output json` response. Currently supported:

| Field  | jq path                  | Typical item type |
|--------|--------------------------|-------------------|
| `note` | `.item.content.note`     | Secure note       |

Unsupported field values cause the action to fail with an explicit error. To add a new field type, add a `case` branch in the action script with the corresponding jq path.

## Outputs

- `secret`: resolved secret value, **base64-encoded**.

All output values are base64-encoded to safely pass multiline secrets (such as kubeconfigs or certificates) through `GITHUB_OUTPUT`. Decode in your workflow:

```yaml
- run: echo "${{ steps.<step-id>.outputs.secret }}" | base64 -d > secret.txt
```

## Requirements and assumptions

- The workflow runner can reach Proton services over the network.
- `pass-cli` and `jq` are installed and available in `PATH` before this action runs.
  - Recommended: use `peinser/actions/proton-pass-install` first.
  - `jq` is preinstalled on GitHub hosted runners.
- The PAT grants access to the target vault/item/field.
- When using `vault_share_id`, the Share ID must belong to the PAT session (each user/PAT gets its own Share ID for a vault).
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

### Finding the correct Share ID for your PAT

Each user (or PAT) gets its own Share ID for a shared vault. To find the Share ID your PAT has access to, run:

```bash
PROTON_PASS_PERSONAL_ACCESS_TOKEN="<your-pat>" pass-cli login
pass-cli vault list --output json
```

Use the `share_id` from the output, not the one from your personal account.

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

      - name: Use resolved secret
        run: |
          SERVICE_API_TOKEN="$(echo "${{ steps.proton_secret.outputs.secret }}" | base64 -d)"
          test -n "${SERVICE_API_TOKEN}"
          echo "Secret was resolved and decoded"
```

Using `vault_share_id` + `item_id`:

```yaml
- name: Resolve secret by share and item ID
  id: proton_secret
  uses: peinser/actions/proton-pass-secret@v1
  with:
    personal-access-token: ${{ secrets.PROTON_PASS_PAT }}
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
    vault_share_id: kxMSS9Ok_rQjUdXujFCR2w5w43oEtH9bGpq5gvMry_buisSpwbzfdiWdIrJJ7_fbf8a1xFjFzXcku1VmTPSfIQ==
    item_title: KUBECONFIG
    field: note

- name: Deploy with resolved kubeconfig
  run: |
    echo "${{ steps.proton_secret.outputs.secret }}" | base64 -d > kubeconfig
    export KUBECONFIG=$(pwd)/kubeconfig
    kubectl get nodes
```

## Security notes

- Prefer PAT login for CI; avoid username/password automation where possible.
- Keep PAT scope narrow (least privilege).
- Never print secret outputs directly to logs.
- Use GitHub encrypted secrets for PAT storage.
- The PAT's Share ID differs from your personal account's Share ID for the same vault. Always verify with `pass-cli vault list`.