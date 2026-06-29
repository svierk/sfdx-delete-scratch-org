# 🗑️ SFDX Delete Scratch Org

This repository implements a simple GitHub composite action for deleting a Salesforce scratch org. It is the natural counterpart to [sfdx-create-scratch-org](https://github.com/svierk/sfdx-create-scratch-org) and is typically used as a cleanup step so that scratch orgs created during a CI run are released again, even if previous steps fail.

## Usage

The action pairs nicely with the `username` output of the create action and an `if: always()` condition to guarantee cleanup:

```yaml
jobs:
  scratch-org-ci:
    name: Scratch Org CI
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Install SF CLI
        uses: svierk/sfdx-cli-setup@main

      - name: Authorize DevHub
        uses: svierk/sfdx-login@main
        with:
          sfdx-url: ${{ secrets.SFDX_AUTH_URL }}
          set-default-dev-hub: true
          alias: devhub

      - name: Create Scratch Org
        id: scratch
        uses: svierk/sfdx-create-scratch-org@main
        with:
          target-dev-hub: devhub
          definition-file: config/project-scratch-def.json
          duration-days: 1

      # ... deploy metadata, run tests, etc. ...

      - name: Delete Scratch Org
        if: always()
        uses: svierk/sfdx-delete-scratch-org@main
        with:
          target-org: ${{ steps.scratch.outputs.username }}
```

## Inputs

| Name         | Required | Default | Description                                                                  |
| ------------ | -------- | ------- | ---------------------------------------------------------------------------- |
| `target-org` | no       |         | Username or alias of the scratch org to delete. Not required if the default org is set. |

The action always runs with `--no-prompt`, so it never blocks the pipeline waiting for confirmation.

## References

The deletion option supported by this GitHub composite action can be found in the Salesforce CLI Command Reference here:

- [org delete scratch](https://developer.salesforce.com/docs/atlas.en-us.sfdx_cli_reference.meta/sfdx_cli_reference/cli_reference_org_commands_unified.htm#cli_reference_org_delete_scratch_unified)

## Releases

Latest release notes can be found on the [release page](https://github.com/svierk/sfdx-delete-scratch-org/releases).

## License

The scripts and documentation in this project are released under the [MIT License](https://github.com/svierk/sfdx-delete-scratch-org/blob/main/LICENSE).
