# OpenTelemetry to Langfuse Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Document how to route OpenClaw.NET Gateway traces directly to Langfuse through the existing OTLP exporter.

**Architecture:** Keep the existing OpenTelemetry exporter registrations and configure only the traces signal with standard signal-specific OTLP environment variables. Add matching English and Simplified Chinese operator guides and link them from the corresponding documentation indexes.

**Tech Stack:** Markdown, existing .NET OpenTelemetry OTLP exporter, Docker environment variables, PowerShell.

## Global Constraints

- Do not change gateway code, package references, or default telemetry behavior.
- Langfuse receives traces only; existing logs and metrics exporters keep their current configuration and destination.
- Secrets must be supplied at runtime and must not be committed to the repository.
- This setup adds no runtime code or dependencies and remains compatible with both NativeAOT and JIT builds.
- Export or authentication failures retain the existing exporter behavior; the guide will not promise a fallback or delivery guarantee.

---

### Task 1: Document direct Langfuse trace export

**Files:**
- Create: `docs/OBSERVABILITY.md`
- Create: `docs/zh-CN/OBSERVABILITY.md`
- Modify: `docs/README.md`
- Modify: `docs/zh-CN/SITE_MAP.md`

**Interfaces:**
- Consumes: Existing Gateway `AddOtlpExporter()` registrations and Langfuse's OTLP ingestion contract.
- Produces: English and Chinese operator instructions using `OTEL_EXPORTER_OTLP_TRACES_ENDPOINT`, `OTEL_EXPORTER_OTLP_TRACES_PROTOCOL`, and `OTEL_EXPORTER_OTLP_TRACES_HEADERS`.

- [x] **Step 1: Write the English operator guide**

Create `docs/OBSERVABILITY.md`. Explain that the Gateway already exports logs, metrics, and traces using OTLP and that the signal-specific trace settings below route only traces to Langfuse. The examples must set the endpoint and protocol directly in the shell and generate the authenticated header at runtime; do not show `${AUTH_STRING}` as a static environment value because the process will not expand it automatically.

Use the EU endpoint `https://cloud.langfuse.com/api/public/otel/v1/traces` and protocol `http/protobuf` in the examples.

Document the EU Cloud endpoint above, plus the US (`https://us.cloud.langfuse.com/api/public/otel/v1/traces`), Japan (`https://jp.cloud.langfuse.com/api/public/otel/v1/traces`), and HIPAA (`https://hipaa.cloud.langfuse.com/api/public/otel/v1/traces`) endpoints. Document self-hosted endpoint format `https://<langfuse-host>/api/public/otel/v1/traces` (local self-hosted Langfuse requires v3.22.0 or newer). State that Langfuse accepts OTLP HTTP with protobuf or JSON but not gRPC, and link to Langfuse's OpenTelemetry ingestion documentation.

Explain that `AUTH_STRING` is the Base64 encoding of `public-key:secret-key`. Include these shell commands to compute and set the header without printing the encoded value:

```bash
export AUTH_STRING="$(printf '%s' "${LANGFUSE_PUBLIC_KEY}:${LANGFUSE_SECRET_KEY}" | base64 | tr -d '\n')"
export OTEL_EXPORTER_OTLP_TRACES_HEADERS="Authorization=Basic ${AUTH_STRING},x-langfuse-ingestion-version=4"
```

Also include this PowerShell equivalent:

```powershell
$authString = [Convert]::ToBase64String([Text.Encoding]::UTF8.GetBytes("$env:LANGFUSE_PUBLIC_KEY`:$env:LANGFUSE_SECRET_KEY"))
$env:OTEL_EXPORTER_OTLP_TRACES_HEADERS = "Authorization=Basic $authString,x-langfuse-ingestion-version=4"
```

Include a PowerShell gateway launch example and a Docker Compose example that passes all three trace variables at runtime. For Docker Compose, show `docker compose run --rm` with `--env` for each variable, relying on the operator to set those variables in the calling shell first; note that the existing Compose service also requires `MODEL_PROVIDER_KEY` and `OPENCLAW_AUTH_TOKEN`. Warn that Base64 is not encryption and the secret key must be protected.

- [x] **Step 2: Write the matching Simplified Chinese guide**

Create `docs/zh-CN/OBSERVABILITY.md` with the same supported configuration, endpoint paths, credential construction, protocol, trace-only routing behavior, Docker and PowerShell examples, Langfuse documentation link, and secret-handling warning. Translate the explanation naturally; keep environment-variable names, endpoint values, and version requirements identical to the English guide.

- [x] **Step 3: Add both guides to their documentation indexes**

In `docs/README.md`, add an observability guide row to the `Using It` table linking to `OBSERVABILITY.md`. In `docs/zh-CN/SITE_MAP.md`, add an observability entry under `运维` linking to `OBSERVABILITY.md`.

- [x] **Step 4: Validate the documentation**

Run:

```powershell
git diff --check
```

Expected: exit code 0 and no whitespace errors.

Manually confirm that both index links resolve and that both guides use the exact Langfuse endpoint path, `http/protobuf`, Basic authentication header, and `x-langfuse-ingestion-version=4`. Confirm the examples do not include real credentials and explain that logs and metrics keep their existing destination. No code tests are required because this task changes documentation only.

- [x] **Step 5: Commit the documentation**

```powershell
git add -- docs/OBSERVABILITY.md docs/zh-CN/OBSERVABILITY.md docs/README.md docs/zh-CN/SITE_MAP.md docs/superpowers/plans/2026-10-09-otel-langfuse.md
git commit -m "docs: document Langfuse OTLP tracing" -m "Co-authored-by: Copilot <223556219+Copilot@users.noreply.github.com>"
```

## Self-Review

- Spec coverage: bilingual operator instructions, trace-specific endpoint/protocol/headers, regional and self-hosted endpoint guidance, runtime-only secrets, index links, preserved logs/metrics routing, and unchanged AOT/JIT behavior are all covered by Task 1.
- Scope: no exporter code, package, test, or application configuration changes are planned.
- Credential handling: examples compute or pass runtime values and explicitly state that Base64 does not protect the underlying secret.
