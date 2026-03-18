<p align="center">
  <strong>▸ RuneFile</strong><br>
  <em>Prompts as code.</em>
</p>

<p align="center">
  A Markdown-based format for parameterized, portable, executable AI prompts.<br>
  Write once. Run anywhere. Version everything.
</p>

<p align="center">
  <a href="./BLUEPRINT.md">Blueprint v0.2</a> ·
  <a href="#quick-start">Quick Start</a> ·
  <a href="#examples">Examples</a> ·
</p>

---

## Why

We've been working with public sector teams adopting AI — and we kept seeing the same thing: prompts living in Google Docs, Slack messages, sticky notes on monitors, email threads titled "final prompt v3 FINAL (2)".

These teams are doing real work — procurement analysis, citizen comms, policy drafting — and their prompts are engineering artifacts. But there's no way to version them, validate them, share them, or run them reliably.

We built RuneFile to fix that.

A `.rune.md` file is just Markdown. Anyone can read it, anyone can write it. But under the hood it's a fully typed, parameterized, composable prompt that a runner can parse and execute against any model.

No vendor lock-in. No proprietary platform. Just files in a repo.

## What is it

RuneFile is an **open format** for AI prompts that combines:

- **Markdown** you can read and write without learning anything new
- **YAML frontmatter** for model config, typed inputs, and output schemas
- **Template variables** like `{{company_name}}` for parameterization
- **Ordered sections** (`## System`, `## User`, `## Assistant`, `## Prompt`) that map directly to API messages

A `.rune.md` file is a **prompt-as-function**: it declares its inputs, defines its behavior, and produces a structured output.

## Quick Start

Here's a complete RuneFile:

```markdown
---
name: summarizer
model: claude-sonnet-4
temperature: 0.3

inputs:
  - name: document
    type: text
    required: true
  - name: length
    type: enum
    options: [brief, detailed]
    default: brief
---

## System
You are a concise, accurate summarizer.

## Prompt
Summarize the following document ({{length}}):

{{document}}
```

Save it as `summarizer.rune.md` and run it:

```bash
rune run summarizer.rune.md \
  --var document="$(cat report.txt)" \
  --var length="brief"
```

## Examples

**Competitive analysis** with structured JSON output:

```bash
rune run analysis.rune.md \
  --var company="Acme" \
  --var industry="climatetech"
```

**Multi-turn translation** with few-shot examples:

```bash
rune run translate.rune.md \
  --var source_lang="English" \
  --var target_lang="Spanish" \
  --var text="The quarterly results exceeded expectations."
```

**Code review** piping source files:

```bash
cat auth.py | rune run review.rune.md \
  --var language="python" \
  --var focus="security" \
  --stdin code
```

**Preview** rendered messages without executing:

```bash
rune render analysis.rune.md \
  --var company="Test" --var industry="saas"
```

See the [`examples/`](./examples) folder for complete, runnable files — or read the [Blueprint](./BLUEPRINT.md) for the full specification.

## Features

| | |
|---|---|
| **Typed inputs** | `string`, `text`, `number`, `boolean`, `enum`, `array`, `file` — with defaults, constraints, and descriptions |
| **Multi-turn** | Ordered `## User` / `## Assistant` sections for few-shot patterns and conversation flows |
| **Composition** | `include` partials to reuse system prompts and build prompt libraries from composable pieces |
| **Output schemas** | Declare JSON Schema for structured outputs — runners validate automatically |
| **Input sanitization** | Template injection protection, enum validation, length limits, strict mode |
| **Tool definitions** | Define function-calling tools in readable Markdown |
| **Provider-agnostic** | Informational `model` field — run against any LLM provider |
| **Git-native** | Plain text files with semver — diff, review, branch, and merge your prompts |

## The Format

```
.rune.md
```

```
---
<YAML frontmatter: name, model, inputs, output>
---

## System
<system instructions>

## User
<few-shot example input>

## Assistant
<few-shot example output>

## Prompt
<parameterized user prompt with {{variables}}>
```

That's it. Markdown you already know, structured for machines to parse.

## CLI (Coming Soon)

We're building the `rune` CLI — a lightweight runner that parses `.rune.md` files and executes them against multiple providers.

```bash
rune run <file> --var key=value     # Execute a prompt
rune render <file> --var key=value  # Preview rendered messages
rune validate <file>                # Validate without executing
rune inspect <file>                 # List inputs and metadata
```

Multi-provider support (Anthropic, OpenAI, Mistral, and others) out of the box.

Star the repo to get notified when it drops.

## Roadmap

- [ ] `rune` CLI with multi-provider support
- [ ] Prompt registries — `rune install @woza/competitor-analysis`
- [ ] Testing framework — `rune test file.rune.md --cases tests.yaml`
- [ ] IDE extensions — syntax highlighting, autocomplete, live preview
- [ ] Execution tracing — structured logs for debugging and auditing
- [ ] Multi-step chains — sequential prompt execution with output piping

## Contributing

RuneFile is an open format. We'd love your input:

- **Read the [Blueprint](./BLUEPRINT.md)** for the full spec
- **Open an issue** for bugs, questions, or ideas
- **Submit a PR** if you want to improve the spec or build tooling
- **Build a runner** in your language of choice

## License

MIT

---

<p align="center">
  <sub>Crafted with ♥ & code by <a href="https://www.wozalabs.com">Woza Labs</a></sub>
</p>
