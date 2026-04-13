---
title: "Self-hosting LLMs for Lead Scoring with Ollama"
description: "Moving from Claude Haiku to local inference: Docker architecture, GPU passthrough, and resource management."
pubDate: "2026-04-13"
tags: ["ollama", "llm", "docker", "infrastructure"]
draft: true
---

Group Scout began as a cloud-first pipeline. Every building permit went to Claude Haiku for scoring. At $0.001 per call, it was cheap; however, as the pipeline grew to thousands of permits, I moved the inference to a local system.

This post explains how I integrated Ollama as a sidecar container.

## The Sidecar Architecture

Ollama runs in the same Docker Compose stack as the Go binary. It exposes an HTTP API for the enrichment layer.

```
┌─────────────────────────────────────────────┐
│  Docker Compose stack                       │
│                                             │
│  groupscout  ──HTTP──▶  ollama:11434        │
│  (Go binary)            (LLM runtime)       │
│                              │              │
│                         named volume        │
│                         ollama_data/        │
│                         (models on disk)    │
└─────────────────────────────────────────────┘
```

A named Docker volume ensures that models persist across restarts. This is critical because Llama 3.1 8B is a five-gigabyte download.

## Docker Configuration

I added the `ollama` service with resource limits to prevent it from starving the Go application.

```yaml
services:
  ollama:
    image: ollama/ollama:latest
    container_name: groupscout_ollama
    restart: unless-stopped
    volumes:
      - ollama_data:/root/.ollama
    networks:
      - groupscout_net
    mem_limit: 12g
    cpus: "2.0"
    healthcheck:
      test: ["CMD", "ollama", "list"]
      interval: 30s
      timeout: 10s
      retries: 5
```

## GPU Passthrough in WSL2

On a Windows 11 machine with an NVIDIA RTX 4060 Ti, CPU-only inference was slow. Enabling GPU passthrough dropped inference time to 1.5 seconds.

The configuration uses the `reservations` block:

```yaml
deploy:
  resources:
    reservations:
      devices:
        - driver: nvidia
          count: 1
          capabilities: [gpu]
```

## Why This Matters

Moving to a local LLM removes per-call costs and API latency. It also allows the pipeline to run offline, fitting the 'home server' ethos. Group Scout uses an environment variable to toggle between local development and local inference.
