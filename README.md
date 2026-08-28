# Workday Extend GitHub Actions demo

This repository contains a conservative Workday Extend CI/CD workflow using
WDCLI. Pull requests and pushes validate and package the app without changing
Workday. An App Hub upload is available only through a manual workflow run with
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
- `WORKDAY_APP_DIR`: Relative app directory; use `.` when the app is at the
  repository root.
- `EXPECTED_WDCLI_VERSION`: Optional; defaults to `1.8.43`.

### Repository secrets

- `WDCLI_CLIENT_ID`: WDCLI system-user client ID.
- `WDCLI_CLIENT_SECRET`: WDCLI system-user client secret.

Do not commit these credentials or a WDCLI authentication/configuration file.

## Required runner

Register a private Windows self-hosted GitHub Actions runner with these labels:

```text
self-hosted, windows, x64, workday-wdcli
```

Install WDCLI `1.8.43` on the runner and add it to `PATH`. Verify it with:

```powershell
wdcli version
```

Do not run pull requests from untrusted forks on this runner. WDCLI persists
authentication locally, so the workflow serializes all WDCLI jobs and logs out
after every run.

## Workday setup

1. Create a WDCLI system user in the Workday Developer Site.
2. Grant the system user access to the target app and account/company.
3. Copy its client ID and secret into the GitHub repository secrets.
4. Add the Workday app source to this repository.
5. Open a pull request to run validation and dry-run packaging.
6. After merging to `main`, manually run the workflow with
   `upload_to_apphub: true` only when an App Hub build is intended.

Tenant deployment requires separate tenant authentication and an explicit app
version or version ID. It is intentionally outside this workflow.
