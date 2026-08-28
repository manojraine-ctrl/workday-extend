# Workday Extend GitHub Actions demo

This repository contains a conservative Workday Extend CI/CD workflow using
WDCLI. Pushes to `main` validate and package the app without changing Workday.
An App Hub upload is available only through a manual workflow run with
`upload_to_apphub` set to `true`.

The workflow deliberately does not deploy to a Workday tenant.

## Repository layout

Place a complete Workday Extend app at the repository root, or set the
`WORKDAY_APP_DIR` repository variable to its relative directory. The app must
include `appManifest.json` and its normal source directories such as
`attributes`, `cards`, `model`, `orchestration`, and `presentation`.

## Required GitHub configuration

### Repository variables

- `WDCLI_ACCOUNT`: Workday Developer Site account/company short ID.
- `WORKDAY_APP_DIR`: Relative directory containing the retrieved app. It
  defaults to `.` when the app is at the repository root.
- `EXPECTED_WDCLI_VERSION`: Optional; defaults to `1.8.43`.

### Temporary authentication model

The workflow temporarily uses an interactive Developer Site login already saved
on the self-hosted runner. It does not use `WDCLI_CLIENT_ID` or
`WDCLI_CLIENT_SECRET`.

Before starting the runner, sign in from a terminal running as the same Windows
user that will run the GitHub Actions runner:

```powershell
wdcli auth login
wdcli account switch qjbwkc
wdcli whoami --format json
```

The saved WDCLI session is tied to that Windows user profile. If the runner is
installed as a Windows service under another account, repeat the interactive
login under that service account. The workflow verifies the user and account
before every Workday operation and does not log out, so that the saved session
remains available for later jobs.

Pull-request execution is disabled while this personal session is used. Restore
PR validation only after migrating back to a least-privilege WDCLI system user.

## Required runner

Register a private Windows self-hosted GitHub Actions runner with these labels:

```text
self-hosted, windows, x64, workday-wdcli, workday-developer-session
```

Install WDCLI `1.8.43` on the runner and add it to `PATH`. Verify it with:

```powershell
wdcli version
```

Do not run pull requests from untrusted forks on this runner. WDCLI persists
authentication locally, so the workflow serializes all WDCLI jobs. This
interactive-session model is temporary; replace it with a least-privilege WDCLI
system user before using the workflow as an unattended or production pipeline.

## Workday setup

1. Confirm the Developer Site user has access to the target app and account.
2. Log in interactively with WDCLI under the runner's Windows identity.
3. Add the Workday app source to this repository.
4. Push an approved change to `main` to run validation and dry-run packaging.
5. After validation, manually run the workflow with
   `upload_to_apphub: true` only when an App Hub build is intended.

Tenant deployment requires separate tenant authentication and an explicit app
version or version ID. It is intentionally outside this workflow.
