# OpenTelemetry and Langfuse

The Gateway already exports logs, metrics, and traces through OpenTelemetry Protocol (OTLP). Configure the trace-specific OTLP environment variables below to send traces to Langfuse while leaving logs and metrics on their existing destinations.

Langfuse accepts OTLP over HTTP with protobuf or JSON; it does not currently support gRPC. This guide uses HTTP/protobuf. For Langfuse Cloud, choose the endpoint for your data region:

| Region | Trace endpoint |
| --- | --- |
| EU | `https://cloud.langfuse.com/api/public/otel/v1/traces` |
| US | `https://us.cloud.langfuse.com/api/public/otel/v1/traces` |
| Japan | `https://jp.cloud.langfuse.com/api/public/otel/v1/traces` |
| HIPAA | `https://hipaa.cloud.langfuse.com/api/public/otel/v1/traces` |

For a self-hosted Langfuse deployment, use `https://<langfuse-host>/api/public/otel/v1/traces`. Local self-hosted ingestion requires Langfuse v3.22.0 or newer.

## Configure from Bash

Set the public key and enter the secret key without echoing it. The Basic authentication value is Base64 of `public-key:secret-key`; Base64 is not encryption.

```bash
export OTEL_EXPORTER_OTLP_TRACES_ENDPOINT="https://cloud.langfuse.com/api/public/otel/v1/traces"
export OTEL_EXPORTER_OTLP_TRACES_PROTOCOL="http/protobuf"

read -r -p "Langfuse public key: " LANGFUSE_PUBLIC_KEY
read -r -s -p "Langfuse secret key: " LANGFUSE_SECRET_KEY
printf '\n'
AUTH_STRING="$(printf '%s' "${LANGFUSE_PUBLIC_KEY}:${LANGFUSE_SECRET_KEY}" | base64 | tr -d '\n')"
export OTEL_EXPORTER_OTLP_TRACES_HEADERS="Authorization=Basic ${AUTH_STRING},x-langfuse-ingestion-version=4"
unset AUTH_STRING LANGFUSE_PUBLIC_KEY LANGFUSE_SECRET_KEY
```

The ingestion-version header enables real-time ingestion in Langfuse v4. Keep the resulting header private: it contains the reversible encoding of the secret key.

Run the gateway in the same shell:

```bash
dotnet run --project src/OpenClaw.Gateway -c Release
```

Or pass the variables to the existing Docker Compose service from that shell:

```bash
docker compose run --rm \
  --env OTEL_EXPORTER_OTLP_TRACES_ENDPOINT \
  --env OTEL_EXPORTER_OTLP_TRACES_PROTOCOL \
  --env OTEL_EXPORTER_OTLP_TRACES_HEADERS \
  openclaw
```

The Compose service also requires its normal `MODEL_PROVIDER_KEY` and `OPENCLAW_AUTH_TOKEN` configuration.

## Configure from PowerShell

Enter the secret as a secure prompt. It is briefly converted in memory to construct the Basic authentication header, so protect the shell and avoid printing environment variables.

```powershell
$env:OTEL_EXPORTER_OTLP_TRACES_ENDPOINT = "https://cloud.langfuse.com/api/public/otel/v1/traces"
$env:OTEL_EXPORTER_OTLP_TRACES_PROTOCOL = "http/protobuf"

$publicKey = Read-Host "Langfuse public key"
$secureSecret = Read-Host "Langfuse secret key" -AsSecureString
$secret = [System.Net.NetworkCredential]::new("", $secureSecret).Password
$authString = [Convert]::ToBase64String([Text.Encoding]::UTF8.GetBytes("${publicKey}:$secret"))
$env:OTEL_EXPORTER_OTLP_TRACES_HEADERS = "Authorization=Basic $authString,x-langfuse-ingestion-version=4"
Remove-Variable secureSecret, secret, authString, publicKey
```

Run the gateway in the same PowerShell session:

```powershell
dotnet run --project src/OpenClaw.Gateway -c Release
```

Langfuse receives traces only. The Gateway's existing logs and metrics exporters keep their current OTLP configuration and destination. Without the trace-specific variables, the existing telemetry behavior is unchanged.

For current endpoint, authentication, and ingestion requirements, see [Langfuse's OpenTelemetry documentation](https://langfuse.com/integrations/native/opentelemetry).
