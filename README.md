# github-settings

Manages the GitHub settings of all repositories in the **pgvillage**, **pgvillage-tools**
and **pgvillage-build** organizations from one place, with
[OpenTofu](https://opentofu.org) and the
[`integrations/github`](https://registry.terraform.io/providers/integrations/github/latest/docs)
provider.

There are too many repositories across too many organizations to keep their settings
consistent by hand. This repository holds one desired state; a pipeline applies it and
corrects drift.

> **Status:** early design. See [#1](https://github.com/pgvillage/github-settings/issues/1)
> for the full design and plan.

## What is managed

- Repository settings: merge methods (squash only), auto-merge, delete branch on merge.
- A ruleset `main` on the default branch of every repository:
  - pull requests with 1 approval, stale approvals dismissed on push, last-push approval,
    code owner review;
  - required status checks, including `Validate PR Title`;
  - a merge queue (squash) wherever it works (public repositories).

The standard is the same for every repository. Organizations and groups of repositories
have a *profile* that mainly differs in the required checks.

Files inside the repositories (`CODEOWNERS`, `pr-title-check.yml`, `merge_group` triggers
in workflows) are **not** managed here: changing them through OpenTofu would push straight
to `main` and bypass review. They are added through pull requests instead.

## How it works

| Trigger               | Job                                                       |
|-----------------------|-----------------------------------------------------------|
| Pull request          | `tofu plan`, posted as a PR comment (read-only access)    |
| Merge to `main`       | `tofu apply` (environment `production`, GitHub App)       |
| Schedule (daily)      | `tofu plan`; on drift, `tofu apply` to restore the config |

- **State** is kept, encrypted (OpenTofu state encryption, AES-GCM), on the separate
  branch `tofu-state`, so `main` only holds configuration.
- **Locking**: all OpenTofu jobs share one concurrency group, so only one run at a time
  writes the state.
- **Access**: only `apply`, running from `main` in the protected `production`
  environment, gets the credentials of the GitHub App that has admin rights on the
  organizations. Pull requests never see them.

## Changing settings

Open a pull request that changes the configuration. Review the `tofu plan` comment;
after the merge the pipeline applies the change.

Changes made by hand in the GitHub UI are reverted by the daily drift check.

## License

[Apache License 2.0](LICENSE)
