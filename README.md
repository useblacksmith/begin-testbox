# begin-testbox

First half of the Blacksmith Testbox action pair. This action runs right after `actions/checkout` and before your dependency installation steps. It installs an SSH public key for remote access, ensures `rsync` is available, and phones home to the Testbox API with a `hydrating` status.

Pair this with [useblacksmith/run-testbox](https://github.com/useblacksmith/run-testbox), which runs after your setup steps to signal the testbox is ready and keep the runner alive until idle timeout.

## Usage

```yaml
steps:
  - uses: actions/checkout@v4

  - name: Begin Testbox
    uses: useblacksmith/begin-testbox@v1
    with:
      testbox_id: ${{ inputs.testbox_id }}
      testbox_token: ${{ inputs.testbox_token }}
      idle_timeout: ${{ inputs.idle_timeout }}
      api_url: ${{ inputs.api_url }}
      ssh_public_key: ${{ inputs.ssh_public_key }}

  # --- your setup steps here ---
  - uses: actions/setup-node@v4
  - run: npm ci
  # --- end setup ---

  - name: Run Testbox
    uses: useblacksmith/run-testbox@v1
    with:
      testbox_id: ${{ inputs.testbox_id }}
      testbox_token: ${{ inputs.testbox_token }}
      idle_timeout: ${{ inputs.idle_timeout }}
      api_url: ${{ inputs.api_url }}
```

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `testbox_id` | yes | | Testbox session ID |
| `testbox_token` | yes | | Bearer token for phone-home authentication |
| `idle_timeout` | no | `10` | Idle timeout in minutes |
| `api_url` | yes | | Backend API URL for phone-home callbacks |
| `ssh_public_key` | no | `''` | SSH public key to install for access |
