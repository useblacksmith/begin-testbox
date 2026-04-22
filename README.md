# begin-testbox

First half of the Blacksmith Testbox action pair. This action runs right after `actions/checkout` and before your dependency installation steps. It discovers configuration from the VM metadata service, phones home to the Testbox API with a `hydrating` status, installs the SSH public key returned by the API, and ensures `rsync` is available.

Pair this with [useblacksmith/run-testbox](https://github.com/useblacksmith/run-testbox), which runs after your setup steps to signal the testbox is ready and keep the runner alive until idle timeout.

## Usage

```yaml
steps:
  - uses: actions/checkout@v4

  - name: Begin Testbox
    uses: useblacksmith/begin-testbox@v2
    with:
      testbox_id: ${{ inputs.testbox_id }}

  # --- your setup steps here ---
  - uses: actions/setup-node@v4
  - run: npm ci
  # --- end setup ---

  - name: Run Testbox
    uses: useblacksmith/run-testbox@v2
```

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `testbox_id` | yes* | | Testbox session ID (empty enables validation-only mode) |
| `api_url` | no | (metadata) | Base URL of the Blacksmith API, e.g. `https://api.example.com` (no path). For local or tunnel testing, set to your reachable origin (e.g. ngrok). |
| `installation_model_id` | no | (metadata) | GitHub App installation model id. **Override** when `api_url` points at a dev database that does not match the metadata service (e.g. local Postgres). |
| `phone_home_bearer_token` | no | (metadata) | `Authorization: Bearer` token for `POST /api/testbox/phone-home`. **Override** with a Sanctum personal access token that exists in the **target** API’s database (e.g. dev). Store as a GitHub **secret**. |

\*Required for a real testbox session; leave empty to validate the workflow only.

## Local or tunnel testing (e.g. ngrok)

The runner still loads `begin-testbox` on a real Blacksmith VM, but you can point phone-home at a **dev** API and align auth with that database:

1. Run your API locally and expose it (ngrok, Cloudflare Tunnel, etc.).
2. Warm up a testbox against that backend so `testbox_id` exists in **that** `testbox_histories` table.
3. Set `api_url` to the tunnel origin, `installation_model_id` to the `github_app_installations.id` row your token belongs to, and `phone_home_bearer_token` to a valid `personal_access_tokens` value for that installation in **that** database.

`run-testbox` reads the same `api_url` and token from `/tmp/.testbox/`; no changes needed there.

## How it works

1. For each of `api_url`, `installation_model_id`, and the bearer token: use the **input** if set, otherwise read from the VM metadata service (`backendURL`, `installationModelID`, `stickyDiskToken`).
2. Phones home to `${api_url}/api/testbox/phone-home` with `hydrating` status, authenticated via the bearer token
3. Receives `ssh_public_key` and `idle_timeout` from the API response
4. Writes state to `/tmp/.testbox/` for `run-testbox` to consume
5. Installs the SSH public key and ensures `rsync` is available
