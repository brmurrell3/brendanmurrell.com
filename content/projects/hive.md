---
title: "Hive"
date: 2026-03-01
tags: ["go", "llm", "agents", "nats"]
summary: "Declarative framework for orchestrating LLM agent teams. One binary, no Docker, no Python dependencies."
---

**[GitHub](https://github.com/brmurrell3/hive)**

Declarative orchestration for AI agent teams. Define agents in YAML, connect them over an embedded NATS message bus, and deploy anywhere — from a laptop to an air-gapped data center. One binary, no Docker, no Python dependencies.

### How it works

Agents are defined in YAML with typed capabilities. Each agent gets a sidecar that handles heartbeats, capability registration, and message routing — agents just implement HTTP handlers. The sidecar auto-generates LLM tool definitions from capability schemas, so agents discover and invoke each other without hard-coded integrations.

### Features

- **YAML-driven agent definitions** with typed capability schemas
- **Embedded NATS** message bus for inter-agent communication
- **Firecracker microVM isolation** on Linux (KVM), process backend on macOS
- **Homebrew installable** — `brew install brmurrell3/tap/hive`
- Built in **Go**
