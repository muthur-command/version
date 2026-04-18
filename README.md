# Muthur Command OS — version channel data

This repository publishes channel metadata and signatures for **Supervisor**, **Muthur Command OS** images, and **mcio** pre-pull consumers.

中文文档: [`README.zh-CN.md`](./README.zh-CN.md)

## Published Files

- Channel JSON: `stable.json`, `beta.json`, `dev.json`
- Signature sidecars: `*.sig` (generated for signed files in workflow)
- Static profiles: `apparmor*.txt`

## Workflow Prerequisites

Configure `version/.github/workflows/version.yml` and repository secrets before running release jobs.

### Environment variables (`version.yml` -> `env`)

| Variable | Purpose |
|---|---|
| `MCOS_VERSION_S3_BUCKET` | Target bucket name. If using AList S3 mapping, this must match the exposed bucket name. |
| `MCOS_VERSION_S3_PUBLIC_BASE` | S3-compatible API endpoint (used as `aws s3 --endpoint-url`). |
| `MCOS_VERSION_PUBLIC_BASE` | Public HTTPS base for clients downloading `stable.json` and assets. |

Variable names keep the **`MCOS_*`** prefix for compatibility with existing **Muthur Command OS** CI and infrastructure; values point at your **Muthur Command** deployment (for example `muthur-command.com` endpoints).

> `MCOS_VERSION_S3_PUBLIC_BASE` and `MCOS_VERSION_PUBLIC_BASE` can be the same host, or split (for example: S3 API endpoint + CDN/public domain).

### Required GitHub Actions secrets

- `AWS_ACCESS_KEY_ID`
- `AWS_SECRET_ACCESS_KEY`

Path: **Settings -> Secrets and variables -> Actions -> New repository secret**

If either secret is missing, upload/sign steps fail with credential errors (for example: `Unable to locate credentials`).

## Trigger Behavior (from workflow)

- `push` to `mc` branch (when `*.txt`, `*.json`, `*.png`, or workflow file changes)
- `pull_request` to `mc` branch (lint/prepare path)
- manual `workflow_dispatch` with `files` input

## TLS Troubleshooting

If you see `SSL validation failed` or `TLSV1_UNRECOGNIZED_NAME`:

1. Ensure the host in `MCOS_VERSION_S3_PUBLIC_BASE` presents a cert whose SAN/CN matches that hostname.
2. Ensure reverse proxy SNI is correctly configured.
3. Prefer a publicly trusted certificate (for example, Let's Encrypt).

Temporary lab-only workaround:

- Set repository variable `MCOS_S3_NO_VERIFY_SSL=true` (Actions Variables).
- Workflow will pass `aws --no-verify-ssl`.
- Do **not** keep this enabled in production.
