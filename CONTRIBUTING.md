# Contributing Templates to PeakInfer

This guide explains how to create and contribute insight and optimization templates for PeakInfer.

## Template Types

PeakInfer uses two types of templates:

| Type | Purpose | Location |
|------|---------|----------|
| **Insight Templates** | Detect issues in LLM inference patterns | `insights/` |
| **Optimization Templates** | Recommend specific improvements | `optimizations/` |

## Directory Structure

```
peakinfer_templates/
├── index.yaml                    # Template index
├── insights/
│   ├── cost/                     # Cost-related insights
│   │   ├── prompt-bloat.yaml
│   │   └── cost-concentration.yaml
│   ├── drift/                    # Drift detection
│   │   ├── streaming-drift.yaml
│   │   └── dead-code.yaml
│   ├── performance/              # Performance issues
│   │   └── latency-explainer.yaml
│   └── waste/                    # Resource waste
│       └── overpowered-model.yaml
├── optimizations/
│   ├── smart-model-routing.yaml
│   ├── vllm-migration.yaml
│   └── ...
└── mappings/
    └── insight-to-optimization.yaml
```

## Insight Template Schema

```yaml
# Required fields
id: string                        # Unique identifier (kebab-case)
version: string                   # Semantic version
name: string                      # Human-readable name
description: string               # Brief description
category: cost|drift|performance|waste|reliability
severity: critical|warning|info
tags: string[]                    # Searchable tags

# Match conditions
match:
  scope: callsite|global|joined|envelope
  conditions:
    - field: string               # Field to evaluate
      op: gt|lt|eq|neq|in|exists|ratio_gt|ratio_lt
      value: any                  # Comparison value
      compare_to: string          # For ratio operators

# Output format
output:
  headline: string                # Mustache template
  evidence: string                # Mustache template

# Optional
recommends: string[]              # IDs of optimization templates
```

### Example: Insight Template

```yaml
id: prompt-bloat
version: "1.0"
name: Prompt Bloat Detection
description: Detects high input/output token ratio indicating prompt inefficiency
category: cost
severity: warning
tags:
  - tokens
  - prompt
  - efficiency

match:
  scope: callsite
  conditions:
    - field: usage.tokens_in
      op: ratio_gt
      compare_to: usage.tokens_out
      value: 20

output:
  headline: "{{input_output_ratio}}x more input than output tokens"
  evidence: "{{location}}: Sending {{tokens_in}} tokens, receiving {{tokens_out}}."

recommends:
  - context-window-optimization
  - system-prompt-optimization
```

## Optimization Template Schema

```yaml
# Required fields
id: string                        # Unique identifier
version: string                   # Semantic version
name: string                      # Human-readable name
description: string               # What this optimization does
layer: application|model|serving|infrastructure
impact_type: cost|latency|throughput
effort: low|medium|high
estimated_impact_percent: number  # 0-100

# Applicability conditions
applicable_when:
  - condition: string             # When to recommend

# Implementation details
implementation:
  summary: string                 # Brief description
  steps: string[]                 # Step-by-step guide
  code_example: string            # Optional code sample

# Validation
verify:
  - metric: string                # What to measure
    expected: string              # Expected improvement

# Optional
related: string[]                 # Related optimization IDs
```

### Example: Optimization Template

```yaml
id: smart-model-routing
version: "1.0"
name: Smart Model Routing
description: Route requests to appropriate models based on complexity
layer: application
impact_type: cost
effort: medium
estimated_impact_percent: 40

applicable_when:
  - Using premium models (gpt-4o, claude-3-opus) for simple tasks
  - High volume of short-output requests
  - Mixed complexity workloads

implementation:
  summary: Implement a router that selects models based on task complexity
  steps:
    - Classify incoming requests by complexity (simple/medium/complex)
    - Route simple tasks to gpt-4o-mini or claude-3-5-haiku
    - Route complex tasks to gpt-4o or claude-3-5-sonnet
    - Monitor quality metrics to tune routing thresholds
  code_example: |
    async function routeRequest(input: string) {
      const complexity = await classifyComplexity(input);

      if (complexity === 'simple') {
        return openai.chat.completions.create({
          model: 'gpt-4o-mini',
          messages: [{ role: 'user', content: input }],
        });
      }

      return openai.chat.completions.create({
        model: 'gpt-4o',
        messages: [{ role: 'user', content: input }],
      });
    }

verify:
  - metric: cost_per_request
    expected: "30-50% reduction"
  - metric: quality_score
    expected: "< 5% degradation"

related:
  - context-window-optimization
  - max-tokens-optimization
```

## Match Operators

| Operator | Description | Example |
|----------|-------------|---------|
| `gt` | Greater than | `value: 1000` |
| `lt` | Less than | `value: 100` |
| `eq` | Equals | `value: true` |
| `neq` | Not equals | `value: false` |
| `in` | In array | `value: ["gpt-4o", "claude-3-opus"]` |
| `exists` | Field exists | `value: true` |
| `ratio_gt` | Ratio greater than | `compare_to: usage.latency_p50, value: 5` |
| `ratio_lt` | Ratio less than | `compare_to: envelope.tps_median, value: 0.5` |

## Available Fields

### Callsite Fields

```typescript
{
  id: string;                     // e.g., "src/api/chat.ts:42"
  file: string;                   // e.g., "src/api/chat.ts"
  line: number;                   // e.g., 42
  provider: string | null;        // e.g., "openai"
  model: string | null;           // e.g., "gpt-4o"
  patterns: {
    streaming: boolean;
    batching: boolean;
    retries: boolean;
    caching: boolean;
    fallback: boolean;
  };
  usage?: {                       // Populated with runtime data
    calls: number;
    tokens_in: number;
    tokens_out: number;
    latency_p50: number;
    latency_p95: number;
    latency_p99: number;
  };
}
```

### Global Fields (scope: global)

```typescript
{
  top_callsite_cost_percent: number;  // % of total cost from top callsite
  total_cost_usd: number;             // Total estimated cost
  total_calls: number;                // Total inference calls
}
```

### Joined Fields (scope: joined)

```typescript
{
  codeOnly: Callsite[];           // In code, not in runtime
  runtimeOnly: RuntimeEvent[];    // In runtime, not in code
  matched: EnrichedCallsite[];    // Matched code + runtime
}
```

## Template Variables

Use Mustache syntax in `output.headline` and `output.evidence`:

| Variable | Description |
|----------|-------------|
| `{{location}}` | File:line reference |
| `{{model}}` | Model name |
| `{{provider}}` | Provider name |
| `{{tokens_in}}` | Input tokens |
| `{{tokens_out}}` | Output tokens |
| `{{p50}}` | p50 latency |
| `{{p95}}` | p95 latency |
| `{{p99}}` | p99 latency |
| `{{calls}}` | Number of calls |
| `{{ratio}}` | Computed ratio |
| `{{percent}}` | Computed percentage |
| `{{count}}` | Count value |
| `{{avg_tokens}}` | Average tokens |
| `{{input_output_ratio}}` | Input/output token ratio |

## Testing Your Template

1. Add your template to the appropriate directory
2. Update `index.yaml` with the new template
3. Run template validation:

```bash
npm run test -- tests/template-conformance.test.ts
```

4. Test against sample data:

```bash
peakinfer analyze ./fixtures/demo-project --verbose
```

## Contribution Process

1. **Fork** the repository
2. **Create** your template following the schema
3. **Test** locally with `npm test`
4. **Submit** a pull request with:
   - Template file(s)
   - Updated `index.yaml`
   - Test case if applicable

## Best Practices

1. **Be specific** - Templates should detect clear, actionable issues
2. **Avoid false positives** - Use multiple conditions when needed
3. **Provide evidence** - Output should explain why the issue was flagged
4. **Suggest solutions** - Link to optimization templates via `recommends`
5. **Version carefully** - Increment version when changing match logic
6. **Tag thoroughly** - Tags help users discover relevant templates

## Questions?

Open an issue on GitHub or check the [PeakInfer documentation](https://peakinfer.com/docs).
