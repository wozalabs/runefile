# 📜 RuneFile Blueprint — A Markdown-Based Specification for Parameterized AI Prompts

**Version:** 0.2  
**Status:** Draft  
**Date:** 2026-03-17

---

## 1. Abstract

RuneFile defines a lightweight, portable, and human-readable specification for authoring, sharing, and executing parameterized AI prompts using Markdown.

Prompts are treated as **declarative, reusable functions** combining:

- Markdown for readability
- YAML frontmatter for configuration
- Template variables for parameterization
- Composable sections for multi-turn conversations

---

## 2. Changes from v0.1

| Area | v0.1 | v0.2 |
|------|------|------|
| Input types | `string \| number \| boolean` | + `text`, `enum`, `array`, `file` |
| Sections | `System`, `Prompt` | + `User`, `Assistant`, `Examples`, `Tools`, `Context` |
| Multi-turn | Not supported | Ordered section sequencing |
| Composition | Future work | `include` directive |
| Output | `text \| json` | + `schema` field (JSON Schema) |
| Security | Minimal | Input sanitization spec |
| Metadata | Minimal | + `tags`, `author`, `license` |

---

## 3. File Format

### 3.1 File Extension

```
.rune.md
```

### 3.2 General Structure

```
---
<YAML frontmatter>
---

<Markdown body with ordered sections>
```

---

## 4. Frontmatter Specification

### 4.1 Required Fields

```yaml
name: string          # Unique identifier (slug format)
```

### 4.2 Optional Fields

```yaml
version: string       # Semver: MAJOR.MINOR.PATCH
description: string   # Human-readable description
author: string        # Author name or handle
tags: string[]        # Categorization tags
license: string       # SPDX identifier

model: string         # Target model (informational, not binding)
temperature: float    # 0.0 - 2.0
max_tokens: integer   # Max output tokens
stop: string[]        # Stop sequences

inputs: Input[]       # Parameter definitions
output: Output        # Output format specification
include: string[]     # Paths to composable partials
```

### 4.3 Input Definition

```yaml
inputs:
  - name: string            # Variable name (snake_case)
    type: InputType         # See 4.3.1
    required: boolean       # Default: true
    default: any            # Default value (optional)
    description: string     # Human-readable hint (optional)
    options: any[]          # For enum type only
    items_type: InputType   # For array type only
    min: number             # For number type: minimum value
    max: number             # For number type: maximum value
    max_length: integer     # For string/text: max characters
```

#### 4.3.1 Input Types

| Type | Description | Example |
|------|-------------|---------|
| `string` | Short text (single line) | A name, identifier, slug |
| `text` | Long text (multi-line) | A paragraph, document excerpt |
| `number` | Numeric value (int or float) | `42`, `3.14` |
| `boolean` | True/false | `true` |
| `enum` | One of predefined options | Requires `options` field |
| `array` | List of values | Requires `items_type` field |
| `file` | File reference (path or URI) | For context injection |

### 4.4 Output Definition

```yaml
output:
  format: text | json | markdown
  schema:                    # Optional: JSON Schema for validation
    type: object
    properties:
      summary:
        type: string
      score:
        type: number
    required: [summary]
```

When `format: json` and `schema` is present, the runner SHOULD validate output against the schema.

### 4.5 Composition with `include`

```yaml
include:
  - ./partials/system-consultant.rune.md
  - ./partials/output-format-json.rune.md
```

**Resolution rules:**

1. Included files are merged **before** the main body sections
2. Sections in the main file **override** included sections with the same name
3. Frontmatter from included files is **ignored** (only sections are imported)
4. Circular includes MUST raise an error

---

## 5. Markdown Body

### 5.1 Section Syntax

Sections use level-2 headings. The **order defines message sequence**.

```markdown
## SectionName
```

### 5.2 Standard Sections

| Section | Maps to | Required | Description |
|---------|---------|----------|-------------|
| `## System` | `role: system` | No | System instructions |
| `## User` | `role: user` | No | User message (alternative to Prompt) |
| `## Prompt` | `role: user` | Yes* | Primary user prompt (*or at least one `## User`) |
| `## Assistant` | `role: assistant` | No | Pre-filled assistant response (few-shot) |
| `## Examples` | Expanded | No | Few-shot example block (see 5.3) |
| `## Context` | Prepended | No | Static context injected before Prompt |
| `## Tools` | `tools[]` | No | Tool/function definitions |

### 5.3 Multi-Turn Sequencing

Sections are processed **in document order**. Repeated section types create multiple messages:

```markdown
## System
You are a translator.

## User
Translate: "Hello, how are you?"

## Assistant
"Hola, ¿cómo estás?"

## User
Now translate: "{{input_text}}"
```

This renders to:

```json
[
  {"role": "system", "content": "You are a translator."},
  {"role": "user", "content": "Translate: \"Hello, how are you?\""},
  {"role": "assistant", "content": "\"Hola, ¿cómo estás?\""},
  {"role": "user", "content": "Now translate: \"Goodbye, friend\""}
]
```

### 5.4 Examples Section

The `## Examples` section is syntactic sugar for few-shot patterns:

```markdown
## Examples

**User:** What is 2+2?
**Assistant:** 4

**User:** What is the capital of France?
**Assistant:** Paris
```

The runner expands each `**User:**` / `**Assistant:**` pair into separate messages.

### 5.5 Context Section

`## Context` content is prepended to the first `## User` or `## Prompt` message:

```markdown
## Context
The company was founded in 2019 and operates in 12 countries.

## Prompt
Summarize the key facts about {{company_name}}.
```

Renders the user message as:

```
The company was founded in 2019 and operates in 12 countries.

Summarize the key facts about Acme Corp.
```

### 5.6 Tools Section

Defines tools available to the model (function calling):

```markdown
## Tools

### get_weather
Get current weather for a location.
- **city** (string, required): City name
- **units** (enum: celsius, fahrenheit): Temperature units
```

Runners SHOULD parse this into the provider's tool format.

---

## 6. Variable Interpolation

### 6.1 Syntax

```
{{variable_name}}
```

### 6.2 Filters (Optional)

Runners MAY support filters:

```
{{name | uppercase}}
{{items | join:", "}}
{{description | truncate:200}}
```

### 6.3 Conditionals (Optional)

Runners MAY support conditional blocks:

```
{{#if premium}}
Include detailed analysis with charts.
{{/if}}

{{#each items}}
- {{this}}
{{/each}}
```

### 6.4 Validation Rules

- Variables MUST match declared inputs
- Missing required variables with no default MUST raise an error
- Undeclared variables SHOULD raise a warning

---

## 7. Execution Model

### 7.1 Lifecycle

```
Parse → Resolve Includes → Validate → Render → Execute → Output
```

1. **Parse**: Read frontmatter + sections
2. **Resolve Includes**: Merge partials (depth-first, detect cycles)
3. **Validate**: Check required inputs, types, constraints
4. **Render**: Interpolate variables into sections
5. **Execute**: Send messages to AI provider
6. **Output**: Return raw or validate against schema

### 7.2 Canonical Message Format

```json
{
  "model": "...",
  "temperature": 0.7,
  "max_tokens": 2000,
  "messages": [
    {"role": "system", "content": "..."},
    {"role": "user", "content": "..."},
    {"role": "assistant", "content": "..."},
    {"role": "user", "content": "..."}
  ]
}
```

---

## 8. Security

### 8.1 Input Sanitization

Runners MUST:

- Strip or escape `{{` and `}}` sequences within user-provided variable values to prevent template injection
- Reject inputs exceeding declared `max_length`
- Validate `enum` values against declared `options`

Runners SHOULD:

- Warn when variable content contains instruction-like patterns (e.g., "ignore previous instructions")
- Support a `--strict` mode that rejects suspicious input

### 8.2 Secrets

- Prompt files MUST NOT contain secrets, API keys, or tokens
- Runners MAY support environment variable references: `{{$ENV.API_KEY}}`
- Environment variables MUST NOT be logged or included in traces

---

## 9. Error Handling

| Error | Severity | Behavior |
|-------|----------|----------|
| Missing frontmatter | Fatal | Abort |
| Invalid YAML | Fatal | Abort |
| Missing `## Prompt` or `## User` | Fatal | Abort |
| Missing required input | Fatal | Abort with list of missing inputs |
| Type mismatch | Fatal | Abort with expected vs actual |
| Unknown variable in template | Warning | Log, continue |
| Circular include | Fatal | Abort |
| Output schema validation fail | Warning | Return output + validation errors |

---

## 10. Example: Complete Prompt File

```yaml
---
name: competitor_analysis
version: 1.0.0
description: Analyzes a company's competitive landscape
author: woza
tags: [strategy, analysis, business]

model: claude-sonnet-4-20250514
temperature: 0.5
max_tokens: 4000

inputs:
  - name: company_name
    type: string
    required: true
    description: Name of the company to analyze

  - name: industry
    type: enum
    required: true
    options: [saas, fintech, ecommerce, healthtech, climatetech, other]

  - name: focus_areas
    type: array
    items_type: string
    required: false
    default: [pricing, positioning, product]
    description: Specific areas to focus the analysis on

  - name: depth
    type: enum
    required: false
    default: standard
    options: [quick, standard, deep]

output:
  format: json
  schema:
    type: object
    properties:
      company:
        type: string
      competitors:
        type: array
        items:
          type: object
          properties:
            name: { type: string }
            threat_level: { type: string, enum: [low, medium, high] }
            strengths: { type: array, items: { type: string } }
            weaknesses: { type: array, items: { type: string } }
      recommendations:
        type: array
        items: { type: string }
    required: [company, competitors, recommendations]
---

## System

You are a senior strategy consultant with 20 years of experience in competitive analysis. You provide structured, actionable insights.

## Examples

**User:** Analyze Notion in the saas industry.
**Assistant:** {"company":"Notion","competitors":[{"name":"Confluence","threat_level":"medium","strengths":["Enterprise adoption","Atlassian ecosystem"],"weaknesses":["Complex UX","Slow innovation"]}],"recommendations":["Focus on AI-native features","Expand template marketplace"]}

## Context

Analysis depth: {{depth}}
Focus areas: {{focus_areas}}

## Prompt

Perform a competitive analysis for **{{company_name}}** in the **{{industry}}** industry.

Identify the top 3-5 competitors and for each provide:
- Threat level (low/medium/high)
- Key strengths
- Key weaknesses

Then provide 3-5 strategic recommendations for {{company_name}}.

Respond in the specified JSON format.
```

---

## 11. CLI Reference

```bash
# Run a prompt with variables
rune run competitor_analysis.rune.md \
  --var company_name="Hectareo" \
  --var industry="climatetech" \
  --var depth="deep"

# Validate a prompt file without executing
rune validate competitor_analysis.rune.md

# Dry-run: render template without sending to model
rune render competitor_analysis.rune.md \
  --var company_name="Test" \
  --var industry="saas"

# Run with environment overrides
rune run file.rune.md \
  --model claude-sonnet-4-20250514 \
  --temperature 0.3

# List inputs for a prompt
rune inspect file.rune.md
```

---

## 12. Future Work

- **Prompt registries** — `rune install @woza/competitor-analysis`
- **Testing framework** — `rune test file.rune.md --cases tests.yaml`
- **Execution tracing** — Structured logs for debugging and auditing
- **IDE extensions** — Syntax highlighting, autocomplete, live preview
- **Multi-step chains** — Sequential prompt execution with output piping
- **Provider adapters** — Normalize across OpenAI, Anthropic, Mistral, etc.
