---
title: "Self-hosting LLMs for Lead Scoring with Ollama"
description: "Moving from Claude Haiku to local inference: Docker architecture, GPU passthrough, and resource management."
pubDate: "2026-04-13"
tags: ["ollama", "llm", "docker", "infrastructure"]
draft: true
---

Group Scout started as a cloud-first pipeline. Every building permit was sent to Claude Haiku for scoring. At $0.001 per call, it was cheap—but as the pipeline grew to thousands of permits across multiple cities, I wanted to move the inference local.

This post covers how I integrated Ollama into the Group Scout stack as a sidecar container.

## The Sidecar Architecture

Ollama runs in the same Docker Compose stack as the main Go binary. It exposes an HTTP API that the `enrichment` layer calls.

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

By using a named Docker volume (`ollama_data`), the LLM models (Mistral, Llama 3.1) persist across container restarts. This is critical because Llama 3.1 8B is a ~5GB download.

## Docker Configuration

I added the `ollama` service with specific resource limits to prevent it from starving the main Go app during heavy inference tasks.

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

Since I'm running this on a Windows 11 machine with an NVIDIA RTX 4060 Ti, CPU-only inference was too slow (8-15s per call). Enabling GPU passthrough via the NVIDIA Container Toolkit dropped inference time to ~1.5s.

The configuration in `deploy/docker-compose.yml` uses the `deploy.resources.reservations` block:

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

Moving to a local LLM removes the per-call cost and the latency of hitting an external API. It also allows the pipeline to run entirely offline, which fits the "home server" ethos of the project. Group Scout now uses the `OLLAMA_ENDPOINT` environment variable to toggle between local development and production-grade local inference.
