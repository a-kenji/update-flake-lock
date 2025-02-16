# update-flake-lock

This is a GitHub Action that will update your flake.lock file whenever it is run.

> **NOTE:** As of v3, this action will no longer automatically install Nix to the action runner. You **MUST** set up a Nix with flakes support enabled prior to running this action, or your workflow will not function as expected.

## Example

An example GitHub Action workflow using this action would look like the following:

```yaml
name: update-flake-lock
on:
  workflow_dispatch: # allows manual triggering
  schedule:
    - cron: '0 0 * * 0' # runs weekly on Sunday at 00:00

jobs:
  lockfile:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v2
      - name: Install Nix
        uses: cachix/install-nix-action@v16
        with:
          extra_nix_config: |
            access-tokens = github.com=${{ secrets.GITHUB_TOKEN }}
      - name: Update flake.lock
        uses: DeterminateSystems/update-flake-lock@vX
        with:
          pr-title: "Update flake.lock" # Title of PR to be created
          pr-labels: |                  # Labels to be set on the PR
            dependencies
            automated
```

## Example updating specific input(s)

> **NOTE**: If any inputs have a stale reference (e.g. the lockfile thinks a git input wants its "ref" to be "nixos-unstable", but the flake.nix specifies "nixos-unstable-small"), they will also be updated. At this time, there is no known workaround.

It is also possible to update specific inputs by specifying them in a space-separated list:

```yaml
name: update-flake-lock
on:
  workflow_dispatch: # allows manual triggering
  schedule:
    - cron: '0 0 * * 0' # runs weekly on Sunday at 00:00

jobs:
  lockfile:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v2
      - name: Install Nix
        uses: cachix/install-nix-action@v16
        with:
          extra_nix_config: |
            access-tokens = github.com=${{ secrets.GITHUB_TOKEN }}
      - name: Update flake.lock
        uses: DeterminateSystems/update-flake-lock@vX
        with:
          inputs: input1 input2 input3
```

## Running GitHub Actions CI

GitHub Actions will not run workflows when a branch is pushed by or a PR is opened by a GitHub Action. There are two ways to have GitHub Actions CI run on a PR submitted by this action.

### Without a Personal Authentication Token

Without using a Personal Authentication Token, you can manually run the following to kick off a CI run:

```
git branch -D update_flake_lock_action
git fetch origin
git checkout update_flake_lock_action
git commit --amend --no-edit
git push origin update_flake_lock_action --force
```

### With a Personal Authentication Token

By providing a Personal Authentication Token, the PR will be submitted in a way that bypasses this limitation (GitHub will essentially think it is the owner of the PAT submitting the PR, and not an Action).
You can create a token by visiting https://github.com/settings/tokens and select at least the `repo` scope. Then, store this token in your repository secrets (i.e. 'https://github.com/<USER>/<REPO>/settings/secrets/actions') as `GH_TOKEN_FOR_UPDATES` and set up your workflow file like the following:

```yaml
name: update-flake-lock
on:
  workflow_dispatch: # allows manual triggering
  schedule:
    - cron: '0 0 * * 1,4' # Run twice a week

jobs:
  lockfile:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v2
      - name: Install Nix
        uses: cachix/install-nix-action@v16
      - name: Update flake.lock
        uses: DeterminateSystems/update-flake-lock@vX
        with:
          token: ${{ secrets.GH_TOKEN_FOR_UPDATES }}
```

## Contributing

Feel free to send a PR or open an issue if you find something functions unexpectedly! Please make sure to test your changes and update any related documentation before submitting your PR.

### How to test changes

In order to more easily test your changes to this action, we have created a template repository that should point you in the right direction: https://github.com/DeterminateSystems/update-flake-lock-test-template. Please see the README in that repository for instructions on testing your changes.
