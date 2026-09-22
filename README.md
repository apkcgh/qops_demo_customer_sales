# Consumer Sales (QOps demo): a Qlik app that ships like software

This repository is a live Qlik Cloud app kept under version control with
[QOps](https://qops.datalabsua.com). The app's load script, sheets, master items and variables are
plain text here. Merge a change and GitHub Actions builds it into the running app. Change the app in
Qlik instead, and a second workflow brings that change back here as a pull request.

No screenshots and no "trust me": the diff you read is the change you see in the app.

## The two directions, which is the whole idea

QOps has two opposite commands, and their names do not say so:

| | Direction | Runs as |
|---|---|---|
| **Build** | repository → Qlik | `Release to Qlik Cloud`, on every merge to `main` |
| **Prepare** | Qlik → repository | `Prepare (Qlik to git)`, on demand and each weekday morning |

Running the wrong one does not fail. It succeeds in the direction you did not want and overwrites the
side you meant to keep — so both live in workflows, where which-one-ran is on the record.

## The release, in four steps

`.github/workflows/release.yml` is deliberately boring. Any step failing stops the run:

1. **Connect** — activate the licence, point QOps at the tenant
2. **Build** — write the repository into the app
3. **Reload** — reload its data, and wait for the result
4. **Publish** — move it into the shared space

## Drift, and why the morning run matters

Somebody will edit the app in the Qlik UI. That is not a problem to be prevented; it is a change to be
captured. The Prepare workflow runs each weekday morning, and if the app no longer matches this
repository it opens a pull request containing the difference. Merge it and the repository catches up.
Close it and the next release overwrites the edit. Either is a decision someone made on purpose, which
is the point.

## Running it yourself

Two repository variables and two secrets:

| Name | Kind | What it is |
|---|---|---|
| `QLIK_TENANT` | variable | tenant URL, no trailing slash |
| `QLIK_SPACE` | variable | shared space to publish into |
| `QLIK_APP_ID`, `QLIK_APP_NAME` | variables | publish needs both, and does not read `.qopsconfig` |
| `QLIK_API_KEY` | secret | Qlik Cloud API key |
| `QOPS_LICENSE` | secret | QOps licence key |

`QOPS_RUNNER` (variable) selects the runner: unset means GitHub-hosted, which is fine for a handful of
runs. A licence activation is bound to the machine it was made on, so a fresh hosted runner activates
again every time — for anything regular, point `QOPS_RUNNER` at a self-hosted runner and the activation
is made once.

Locally, the same thing without CI:

```powershell
QOps-SetConfig -Profile demo-cloud -Mode QlikSaaS -SenseURL <tenant> -ApiKeyValue <api-key>
QOps-Prepare -NoData      # bring the app into this folder
QOps-Build                # push this folder into the app
```

## Why this matters

- A BI change becomes a code review: who changed it, what changed, why.
- Rollback is `git revert`, not a restore ticket.
- Promotion across spaces and tenants is the same command with a different filter.

QOps is a commercial PowerShell module by DatalabsUa — 72 cmdlets for versioning Qlik Sense, Qlik Cloud
and QlikView apps. This demo runs QOps 3.0.20.
