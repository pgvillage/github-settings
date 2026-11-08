# github-settings

Manages the GitHub settings of all repositories in the **pgvillage**, **pgvillage-tools** and **pgvillage-build**
organizations from one place, with [ghsettings](https://github.com/pgvillage-tools/ghsettings).

There are too many repositories across too many organizations to keep their settings consistent by hand. This
repository holds the desired state; a pipeline applies it and corrects drift.

> **Status:** being set up. See [#1](https://github.com/pgvillage/github-settings/issues/1) for the design and the
> plan.

## What is managed

- Repository settings, e.g. merge methods (squash only), auto-merge, delete branch on merge.
- A ruleset `main` on the default branch of every repository:
  - pull requests with 1 approval, stale approvals dismissed on push, last-push approval, code owner review,
    resolved review threads;
  - no deletion of, and no force pushes to, the default branch;
  - required status checks: one aggregate CI check (`CI result`) and `Validate PR Title`;
  - a merge queue (squash) wherever it works.

The standard is the same for every repository. Groups of repositories have a *profile* that mainly differs in the
required checks.

Files inside the repositories (`CODEOWNERS`, `pr-title-check.yml`, the `CI result` job, `merge_group` triggers in
workflows) are **not** managed here: they are added through pull requests, so they get reviewed like any other
change.

## Layout

```text
config/
  standard.yaml          # the standard for every repository
  <org>.yaml             # per org: which repositories get which profile
```

See the [ghsettings README](https://github.com/pgvillage-tools/ghsettings#config) for the config format.

## How it works

| Trigger          | Job                                                                  |
|------------------|----------------------------------------------------------------------|
| Pull request     | `ghsettings plan`, posted as a PR comment (read-only GitHub App)     |
| Merge to `main`  | `ghsettings apply` (environment `production`, admin GitHub App)      |
| Schedule (daily) | `ghsettings plan`; on drift, `apply` to restore the config           |

ghsettings is stateless: GitHub itself is the state. Only `apply`, running from `main` in the protected
`production` environment, gets the credentials of the GitHub App with admin rights on the organizations. Pull
requests never see them.

## Changing settings

Open a pull request that changes the configuration. Review the plan in the PR comment; after the merge the pipeline
applies the change.

Changes made by hand in the GitHub UI are reverted by the daily drift check.

## License

[GNU General Public License v3.0](LICENSE)
