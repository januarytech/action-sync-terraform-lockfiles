# `action-sync-terraform-lockfiles`

This GitHub Actions workflow will automatically push a commit to Pull Requests in your Terraform repository to ensure that lockfiles exist for every platform your developers (or Terraform automation runners) use. This both enforces that lockfiles exist _at all_ and provides a performance boost, because given a lockfile for e.g. macOS arm64 (what many developers use), running `terraform init` on Linux amd64 (what your CI system is likely using) performs a lot of time-consuming signature verification work, which is a waste. This workflow lets you do this work once, in Pull Request CI, and then commit it.

Part of the motivation behind this workflow is that when juggling many statefiles - common when infrastructure grows large enough - it's difficult to enforce that lockfiles are committed to version control, for *all* of these statefile root modules. Therefore, this workflow discovers root modules from Terraform automation configuration. Currently only Terrateam is supported, but others may be supported in the future.

## Maintenance disclaimer

This project is offered as-is by January, and is maintained on a best-effort basis.

We'll do our best to merge Pull Requests, triage issues, etc. - but please understand that this isn't our primary focus. If you're submitting a PR that e.g. adds a feature you need, you should expect to maintain a downstream fork for long periods of time.

## Usage

If you're looking at this workflow, you probably already have statefiles with incomplete or outright missing `.terraform.lock.hcl` files. This is what the `selective-update` input ([see below](#selective-input)) is for - you set it to `false` to correct your historic problems, then remove it to improve performance. For example:

```yaml
name: update-terraform-lockfiles
on:
  pull_request:
    types:
      - opened
      - synchronize
      - reopened

jobs:
  update-terraform-lockfiles:
    uses: januarytech/action-sync-terraform-lockfiles/.github/workflows/update-lockfiles.yml@main # Change this to the version you want to pin to
    with:
      selective-update: false
```

Commit this file to `.github/workflows/update-lockfiles.yml` in a new branch, then create a Pull Request. When the workflow has fixed existing problems and the PR is merged, remove the `with:` block and commit.

## Input reference

<!--
grep -v '#' .github/workflows/update-lockfiles.yml | yq '.on.workflow_call.inputs | to_entries[] | "### `" + .key + "`\n\n" + .value.type + ", default `" + .value.default + "`. " + .value.description + "\n"' | sed -e 's/^bool/Bool/' -e 's/^str/Str/' -e 's/^num/Num/'
-->

### `engine-version`

String, default `1.5.7`. The Terraform version to use for module initialization.

### `platforms`

String, default `darwin_arm64,darwin_amd64,linux_amd64`. Comma-separated list of platforms to create lockfiles for.

### `selective-update`

Boolean, default `true`. Whether to selectively update state directories based on files changed in the PR. Set to false to fix historic problems.

### `parallelization-limit`

Number, default `3`. How many statefile directories to update concurrently per Actions runner CPU core. Set to 0 for unlimited (can lead to mysterious problems on standard GitHub-hosted runners).

### `free-disk-space`

Boolean, default `true`. Whether or not to free runner disk space before beginning, since Terraform providers can be very large.

## Author

AJ Jordan <aj.jordan@january.com>, on behalf of [January Technologies](https://www.january.com/) ([we're hiring](https://www.january.com/careers) - come help us fix what's broken in credit!)

## License

MIT
