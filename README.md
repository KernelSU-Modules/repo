# KernelSU Modules — data repo

This repository is the **source of truth** for [KernelSU Modules](https://modules.kernelsu.org):
which module IDs exist, which GitHub repository each ID is bound to, review/moderation state, the maintainer
and ban lists, and the lock file of validated releases.

It contains **no code** and **no module files**. Module sources, docs and ZIPs live in each author's own
GitHub repository; the site and client index are built from this data by the tools in
[`KernelSU-Modules/tools`](https://github.com/KernelSU-Modules/tools).

## Writes: KernelSU Modules Bot only

- Every commit is made by the **KernelSU Modules Bot** GitHub App (via the Git Data API). A ruleset blocks
  direct pushes, force pushes and branch deletion for everyone else, including org owners.
- **Pull requests are not accepted.** Submit or transfer a module at
  <https://developers.kernelsu.org>; maintainers review in the Portal or the Telegram review group.
- `sources.lock.json` and `status.json` are written only by the sync workflow, always together in one commit
  (`Action: sync`, `Actor: app:kernelsu-modules-bot via sync`), and only when something substantive changed
  (no periodic heartbeat commits).

## Layout

| Path | Content | Schema |
| --- | --- | --- |
| `modules/{id}.yaml` | One record per module ID: status, repository binding, open request, last review, moderation | `RegistryRecord` |
| `policy.yaml` | ID rules, reserved IDs/prefixes, size limits, retention | `Policy` |
| `maintainers.yaml` | Maintainers: GitHub user id ↔ Telegram user id | `Maintainers` |
| `banned.yaml` | Banned developers (GitHub user id + reason) | `Banned` |
| `sources.lock.json` | Locked content commit, validated releases, asset sha256, versionCode high-water marks, sync health | `SourcesLock` |
| `status.json` | Compact per-ID summary for the Portal / cron Worker: health, stable, `maxVersionCode`, result of validating the open request | `RegistryStatus` |

JSON Schemas are published in [`KernelSU-Modules/tools`](https://github.com/KernelSU-Modules/tools) under `schemas/`. Validate a checkout with
`pnpm --filter @kernelsu-modules/sync check-data -- --data-dir <path>` from the tools repo.

Record states: `pending → listed | rejected`, `listed ⇄ suspended`, `archived` (author gave up; ID kept).

## Audit trail

Git history is the audit log. Each commit message carries trailers:

```
<subject>

Action: submit|transfer|approve|reject|suspend|unsuspend|archive|ban|unban|maintainers|expire|sync|init
Module: <id>                                   (when applicable)
Submission: <submissionId>                     (when applicable)
Submitted-by: github:<id>
Approved-by: github:<id> via telegram|portal|auto      (approve)
Rejected-by: github:<id> via telegram|portal           (reject)
Actor: github:<id> via portal|telegram|cron            (other actions)
Actor: app:<app-slug> via sync|cron                    (automated writes, e.g. app:kernelsu-modules-bot via sync)
```

Example: `git log --format='%h %s%n%(trailers:key=Action,key=Module,key=Approved-by)' -- modules/demo.toolbox.yaml`.

User identities are GitHub numeric user ids (`github:<id>`); look them up with `gh api user/<id>`.
