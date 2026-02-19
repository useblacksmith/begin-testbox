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
| `testbox_id` | yes | | Testbox session ID |

## How it works

1. Reads `installationModelID`, `backendURL`, and `stickyDiskToken` from the VM metadata service
2. Phones home to `/api/testbox/phone-home` with `hydrating` status, authenticated via the sticky disk token
3. Receives `ssh_public_key` and `idle_timeout` from the API response
4. Writes state to `/tmp/.testbox/` for `run-testbox` to consume
5. Installs the SSH public key and ensures `rsync` is available
