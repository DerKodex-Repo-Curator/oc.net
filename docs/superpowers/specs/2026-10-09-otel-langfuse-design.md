# OpenTelemetry to Langfuse Design

## Goal

Document how operators can send OpenClaw.NET Gateway traces directly to
Langfuse using the OpenTelemetry Protocol (OTLP), without adding a
Langfuse-specific SDK, exporter, dependency, or application configuration.

## Current State

`GatewayTelemetryExtensions.AddGatewayTelemetry` configures OTLP exporters for
logs, metrics, and traces. Exporter configuration can use the standard
signal-specific OpenTelemetry environment variables.

Langfuse accepts OTLP traces over HTTP, not gRPC. Sending only traces to
Langfuse should not change the existing destination for logs and metrics.

## Design

Add English and Simplified Chinese operator documentation with a trace-only
configuration recipe using:

- `OTEL_EXPORTER_OTLP_TRACES_ENDPOINT` for the Langfuse region's trace endpoint.
- `OTEL_EXPORTER_OTLP_TRACES_PROTOCOL=http/protobuf` for the supported OTLP
  transport.
- `OTEL_EXPORTER_OTLP_TRACES_HEADERS` for Basic authentication and the
  `x-langfuse-ingestion-version=4` header.

The guides will cover Langfuse Cloud region endpoints and self-hosted endpoint
format, explain how to form the Basic authentication value from the project
public and secret keys, and provide Docker and PowerShell examples. Secrets
must be supplied at runtime and must not be committed to the repository.

Link the guides from the English and Chinese documentation indexes. Do not
change gateway code, package references, or default telemetry behavior.

## Behavior and Boundaries

- Langfuse receives traces only; existing logs and metrics exporters keep their
  current configuration and destination.
- Without the trace-specific environment variables, the current OTLP behavior
  remains unchanged.
- The setup uses the existing OpenTelemetry exporter and remains compatible
  with both NativeAOT and JIT builds because it adds no runtime code or
  dependencies.
- Export or authentication failures retain the existing exporter behavior; the
  guide will not promise a fallback or delivery guarantee.

## Validation

This documentation-only change needs no code test unless documentation checks
exist. Validate the examples and links, ensure no credentials are present, and
confirm the documented endpoint, protocol, and headers against Langfuse's
current OpenTelemetry ingestion documentation.
