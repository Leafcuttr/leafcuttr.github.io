# HTTP Proxy

The HTTP proxy is an embedded HTTP server that provides a REST API for producing records to Kafka topics.

## Configuration

The HTTP proxy is controlled by the `lc.http.proxy.enable` configuration option, which is disabled by default. When enabled, it starts automatically as part of the Kafka server startup process.

## Core Functionality

The proxy runs on port 8080 by default and accepts HTTP POST requests to produce records to Kafka topics. 

See [HTTP inteface](../interfaces/http.md) for the provided end points.

## Notes

The proxy creates a Kafka producer with `acks=all` and 3 retries for reliability.

The proxy can automatically create topics if they don't exist (when configured).