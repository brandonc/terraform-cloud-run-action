# terraform-cloud-run-action

## Overview

A GitHub Action step that fetches outputs from a terraform cloud workspace.

### Inputs

- `token` (**Required**): The token of the TFC/E instance which holds the workspace that manages your tflocal instance.
- `organization` (**Required**): The TFC/E organization that manages the specified workspace.
- `workspace` (**Required**): The name of the TFC/E workspace that manages the tflocal configuration.
- `hostname` (**Optional**): The hostname of the TFC/E instance which holds the workspace that manages your tflocal instance. Defaults to `app.terraform.io`.
- `wait` (**Optional**): If set, waits for the run to terminate and resources to be processed before the action finishes. Defaults to true.
- `auto-apply` (**Optional**): If set, applies changes when a Terraform plan is successful. Defaults to true.
- `is-destroy` (**Optional**): If set, a destroy plan will be run. Defaults to false.
- `message` (**Optional**): A custom message to associate with the run. Default to "Run created by GitHub action"
- `replace-addrs` (**Optional**): Multi-line list of resource addresses to be replaced. Use one address per line.
- `target-addrs` (**Optional**): Multi-line list of resource addresses that Terraform should focus its planning efforts on. Use one address per line.

[Read more about the Runs API](https://developer.hashicorp.com/terraform/cloud-docs/api-docs/run#create-a-run)

### Outputs

- `run-id`: The run ID for the created run.

You can use this action in conjunction with `brandonc/terraform-cloud-run-action` to create infrastructure and fetch new outputs to help utilize it:

## Example

```yaml
name: Nightly Test
on:
  workflow_dispatch:
  schedule:
    - cron: 0 0 * * *

jobs:
  infra:
    runs-on: ubuntu-latest
    steps:
      - name: Create infra
        id: fetch
        uses: brandonc/terraform-cloud-run-action@v1
        with:
          token: ${{ secrets.TFC_TOKEN }}
          organization: example-org
          workspace: my-tflocal-workspace
          wait: true

  tests:
    runs-on: ubuntu-latest
    needs: [infra]
    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Fetch infra secrets
        id: fetch
        uses: brandonc/terraform-cloud-outputs-action@v1
        with:
          token: ${{ secrets.TFC_TOKEN }}
          organization: example-org
          workspace: my-tflocal-workspace

      - name: Tests
        run: go test ./...
        env:
          SOME_FOO: ${{ fromJSON(steps.fetch.outputs.workspace-outputs-json).foo }}
          SOME_BAR: ${{ fromJSON(steps.fetch.outputs.workspace-outputs-json).bar }}

  cleanup:
    runs-on: ubuntu-latest
    needs: [tests]
    if: "${{ always() }}"
    steps:
      - name: Destroy infra
        uses: brandonc/terraform-cloud-run-action@v1
        with:
          token: ${{ secrets.TFC_TOKEN }}
          organization: example-org
          workspace: my-tflocal-workspace
          is-destroy: true
          wait: true
```
