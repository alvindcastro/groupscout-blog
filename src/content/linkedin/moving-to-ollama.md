---
title: "Moving to local LLMs for scoring leads."
description: "Why I switched from Claude to self-hosted Ollama."
pubDate: "2026-04-13"
draft: false
tags: ["ollama", "llm", "self-hosting", "docker"]
---

At $0.001 per call, Claude Haiku was cheap. But as the GroupScout pipeline grew, I wanted more control.

I moved from cloud-first inference to self-hosted **Ollama**.

Why?

- **Cost:** Free (after hardware).
- **Control:** No external API dependencies or latency spikes.
- **Privacy:** Data never leaves my local stack.

I'm running Ollama as a Docker sidecar container with GPU passthrough on an RTX 4060 Ti. Inference time dropped to ~1.5s per lead.

The best part? It integrates directly into the Go enrichment layer.

Same pipeline. Same accuracy. Zero per-call cost.

#ollama #selfhosting #llms #docker #golang
