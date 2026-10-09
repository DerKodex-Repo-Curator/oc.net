# OpenTelemetry 接入 Langfuse

Gateway 已通过 OpenTelemetry Protocol（OTLP）导出日志、指标和 traces。设置以下 trace 专属 OTLP 环境变量，即可将 traces 发往 Langfuse，同时让日志和指标继续使用当前目标。

Langfuse 支持通过 HTTP 接收 OTLP protobuf 或 JSON，目前不支持 gRPC。本指南使用 HTTP/protobuf。Langfuse Cloud 请按数据区域选择 endpoint：

| 区域 | Trace endpoint |
| --- | --- |
| EU | `https://cloud.langfuse.com/api/public/otel/v1/traces` |
| US | `https://us.cloud.langfuse.com/api/public/otel/v1/traces` |
| 日本 | `https://jp.cloud.langfuse.com/api/public/otel/v1/traces` |
| HIPAA | `https://hipaa.cloud.langfuse.com/api/public/otel/v1/traces` |

自托管 Langfuse 使用 `https://<langfuse-host>/api/public/otel/v1/traces`。本地自托管 ingestion 需要 Langfuse v3.22.0 或更高版本。

## 使用 Bash 配置

设置 public key，并以不回显的方式输入 secret key。Basic authentication 值是 `public-key:secret-key` 的 Base64 编码；Base64 不是加密。

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

`x-langfuse-ingestion-version=4` header 可启用 Langfuse v4 实时 ingestion。生成后的 header 包含 secret key 的可逆编码，必须妥善保管。

在同一个 shell 中启动 Gateway：

```bash
dotnet run --project src/OpenClaw.Gateway -c Release
```

也可以在同一 shell 中将这些变量传给现有 Docker Compose 服务：

```bash
docker compose run --rm \
  --env OTEL_EXPORTER_OTLP_TRACES_ENDPOINT \
  --env OTEL_EXPORTER_OTLP_TRACES_PROTOCOL \
  --env OTEL_EXPORTER_OTLP_TRACES_HEADERS \
  openclaw
```

Compose 服务还需要照常配置 `MODEL_PROVIDER_KEY` 和 `OPENCLAW_AUTH_TOKEN`。

## 使用 PowerShell 配置

通过安全提示输入 secret。构造 Basic authentication header 时会短暂在内存中将其转换为普通字符串；请保护 shell 会话，避免打印环境变量。

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

在同一个 PowerShell 会话中启动 Gateway：

```powershell
dotnet run --project src/OpenClaw.Gateway -c Release
```

Langfuse 只接收 traces。Gateway 现有的日志和指标 exporter 继续使用当前 OTLP 配置和目标。未设置 trace 专属变量时，现有 telemetry 行为保持不变。

endpoint、认证和 ingestion 要求请参阅 [Langfuse OpenTelemetry 文档](https://langfuse.com/integrations/native/opentelemetry)。
