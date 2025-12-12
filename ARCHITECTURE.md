# PeakInfer Templates Architecture

## Philosophy

> "First, show them what's wrong. Then, show them how to fix it."

This repository contains two complementary template types:

1. **Insights** - Detection patterns that reveal problems
2. **Optimizations** - Implementation guides that solve problems

The magic happens when they're linked: an insight doesn't just tell you there's a problem, it points you to the solution.

## Directory Structure

```
peakinfer_templates/
├── README.md                    # Quick start guide
├── ARCHITECTURE.md              # This file
│
├── schema/                      # JSON Schema definitions
│   ├── insight.schema.json      # Validates insight templates
│   └── optimization.schema.json # Validates optimization templates
│
├── insights/                    # Detection patterns
│   ├── manifest.json            # Index of all insights
│   │
│   ├── cost/                    # Cost-related issues
│   │   ├── prompt-bloat.yaml
│   │   ├── retry-explosion.yaml
│   │   ├── cost-concentration.yaml
│   │   └── overpowered-extraction.yaml
│   │
│   ├── drift/                   # Code ↔ Runtime mismatches
│   │   ├── dead-code.yaml
│   │   ├── streaming-drift.yaml
│   │   └── untested-fallback.yaml
│   │
│   ├── performance/             # Speed & throughput issues
│   │   ├── throughput-gap.yaml
│   │   ├── latency-explainer.yaml
│   │   └── context-accumulation.yaml
│   │
│   └── waste/                   # Underutilization
│       ├── overpowered-model.yaml
│       └── token-underutilization.yaml
│
├── optimizations/               # Implementation guides
│   ├── index.yaml               # Index of all optimizations
│   ├── application-layer/
│   ├── serving-layer/
│   ├── infrastructure-layer/
│   └── cross-layer/
│
└── mappings/
    └── insight-to-optimization.yaml  # Links problems → solutions
```

## Template Types

### Insight Templates (Detection)

**Purpose:** Pattern-match against runtime/static data to surface issues.

**Schema:**
```yaml
id: prompt-bloat                          # Unique identifier
version: "1.0"                            # Template version
name: "Prompt Bloat Detection"            # Human-readable name
description: "Detects excessive input tokens relative to output"

# Source attribution (link to blog post, research, etc.)
source:
  url: "https://www.kalmantic.com/posts/system-prompt-optimization..."
  title: "Stop Paying 40x More for Redundant AI Instructions"

# Classification
category: cost                            # cost | drift | performance | waste
severity: warning                         # critical | warning | info
tags: ["tokens", "prompt", "optimization"]

# Detection logic
match:
  scope: callsite                         # callsite | joined | global | envelope
  conditions:
    - field: usage.tokens_in
      op: ratio_gt
      compare_to: usage.tokens_out
      value: 20

# Output when triggered
output:
  headline: "{{ratio}}x more input than output tokens"
  evidence: "{{location}}: {{tokens_in}} in → {{tokens_out}} out"

# Link to solutions
recommends:
  - optimization: context-window-optimization
    relevance: 0.9
    reason: "Reduce input tokens via summarization or sliding window"
  - optimization: semantic-caching
    relevance: 0.7
    reason: "Cache responses to avoid redundant processing"

# Metadata
author: "kalmantic"
created: "2024-12-13"
updated: "2024-12-13"
```

### Optimization Templates (Remediation)

**Purpose:** Step-by-step implementation guides with economics and monitoring.

**Schema:** (existing format in application-layer/, serving-layer/, etc.)

## The Link: Insights → Optimizations

The `mappings/insight-to-optimization.yaml` file connects problems to solutions:

```yaml
# When PeakInfer detects an insight, it can recommend optimizations
mappings:
  prompt-bloat:
    primary: context-window-optimization
    alternatives:
      - semantic-caching
      - smart-model-routing

  dead-code:
    primary: null  # No optimization, just cleanup
    action: "Remove unused callsites"

  throughput-gap:
    primary: vllm-high-throughput-optimization
    alternatives:
      - gptq-4bit-quantization
```

## Future: The Full Loop

```
┌─────────────────────────────────────────────────────────────────┐
│                     PeakInfer Flow                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   peakinfer .                                                   │
│       │                                                         │
│       ▼                                                         │
│   ┌─────────────┐    match    ┌─────────────┐                  │
│   │  Your Code  │────────────▶│  Insights   │                  │
│   │  + Runtime  │             │  Templates  │                  │
│   └─────────────┘             └──────┬──────┘                  │
│                                      │                          │
│                                      ▼                          │
│                              ┌─────────────┐                   │
│                              │  Findings   │                   │
│                              │  Report     │                   │
│                              └──────┬──────┘                   │
│                                      │                          │
│   peakinfer recommend                │ (future)                │
│       │                              ▼                          │
│       │                      ┌─────────────┐                   │
│       └─────────────────────▶│ Optimization│                   │
│                              │  Templates  │                   │
│                              └──────┬──────┘                   │
│                                      │                          │
│                                      ▼                          │
│                              ┌─────────────┐                   │
│                              │   Action    │                   │
│                              │    Plan     │                   │
│                              └─────────────┘                   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## Contributing

1. **New Insight:** Add to `insights/{category}/` with proper schema
2. **New Optimization:** Add to `optimizations/{layer}/`
3. **Link them:** Update `mappings/insight-to-optimization.yaml`

## Versioning

- Templates are individually versioned (`version: "1.0"`)
- `manifest.json` tracks the overall collection version
- Breaking changes bump major version
