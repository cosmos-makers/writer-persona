# writer-persona

A [Claude Code](https://docs.anthropic.com/en/docs/claude-code) skill that extracts your writing persona from real messages, measures its accuracy, and writes in your voice.

## The Problem

You want AI to write as you — in emails, Slack, code reviews. But how close is "close enough"? Without measurement, you're guessing.

## The Solution

**writer-persona** creates a feedback loop: extract, measure, improve, write.

```
Bootstrap (analyze 100+ real messages)
    ↓
Backtest (AI writes drafts → compare with what you actually wrote)
    ↓
Score (8-axis linguistic evaluation)
    ↓
Auto-correct (fix the biggest gaps)
    ↓
Write (generate messages in your voice)
```

The AI never sees your actual response when generating drafts — it only gets the conversation context. This information isolation ensures honest evaluation.

## Commands

| Command | What it does |
|---------|-------------|
| `/writer-persona` | Show persona status and score |
| `/writer-persona --bootstrap` | Extract persona from 100+ real messages |
| `/writer-persona --backtest` | Measure accuracy (last 24h by default) |
| `/writer-persona --backtest --days 7` | Measure over last 7 days |
| `/writer-persona --calibrate` | Manual tuning with your feedback |
| `/writer-persona --write "context"` | Write as you in a given situation |
| `/writer-persona --write` | Interactive: describe the situation, get a draft |

## Key Features

- **8-axis evaluation** grounded in computational stylometry research
- **Self-similarity ceiling** — your own writing varies; the AI is scored against *your* natural range, not perfection
- **Data source agnostic** — Slack, email, Discord, text files, or any MCP tool
- **Language adaptive** — English, Korean, Japanese, or any language
- **Auto-convergence** with overfitting guards (max 2 axes per run, stall detection)
- **Write mode** — the payoff: generate messages that sound like you

## The 8 Axes

| Axis | Weight | What it measures |
|------|--------|-----------------|
| Semantic Fidelity | 25% | Does the draft say the right *thing*? |
| Tone / Sentiment | 20% | Does it *feel* like you? |
| Formality / Register | 15% | Right level of formal/casual? |
| Discourse Patterns | 15% | How you structure responses |
| Lexical Choice | 10% | Your characteristic vocabulary |
| Function Words | 5% | Unconscious grammatical habits |
| Sentence Structure | 5% | Length, complexity patterns |
| Formatting | 5% | Emoji, punctuation, layout |

See [`references/evaluation-rubric.md`](references/evaluation-rubric.md) for the full framework and academic citations.

## Installation

```bash
git clone https://github.com/cosmos-makers/writer-persona.git

# Copy into your Claude Code skills directory
cp -r writer-persona /path/to/your/project/.claude/skills/writer-persona
```

Or manually copy the files into `.claude/skills/writer-persona/` in any project where you use Claude Code.

## Quick Start

### 1. Extract your persona

```
/writer-persona --bootstrap
```

The skill asks you:
- Your name and language
- How to collect your messages (Slack MCP, paste, file, email MCP, etc.)
- Your situation categories (defaults provided)

Then it analyzes 100+ messages and builds a comprehensive writing persona.

### 2. Measure accuracy

```
/writer-persona --backtest
```

Finds recent conversations where you responded, generates AI drafts *without seeing your answers*, and scores them on 8 axes.

### 3. Write in your voice

```
/writer-persona --write "reply to Sarah's Slack message about the API deadline"
```

Or interactively:
```
/writer-persona --write
```
The skill asks who you're writing to, what the situation is, and generates a draft in your style.

## Supported Data Sources

writer-persona is **data source agnostic**. On first run, it asks how you want to provide messages:

| Source | How |
|--------|-----|
| **Slack** | Via MCP tools (e.g., `mcp-slack`) |
| **Email** | Via email MCP tools |
| **Discord** | Via Discord MCP tools |
| **Manual** | Paste messages or provide a file path |
| **Any MCP** | Describe the tool and query pattern |

## How Scoring Works

### Self-Similarity Ceiling

Even you don't write identically every time. A quick "lol nice" and a detailed technical explanation are both authentically you — but they'd score poorly against each other.

The **self-similarity ceiling** measures this natural variation by scoring pairs of your own messages. A typical ceiling is 70-80. This becomes the 100-point baseline.

### Information Isolation

The draft-generating agent **never sees your actual response**. It only receives your persona definition and the conversation context. This ensures the backtest measures real predictive ability, not copying.

## File Structure

```
writer-persona/
├── SKILL.md                    # Main skill definition
├── persona.template.md         # Blank persona template
├── references/
│   └── evaluation-rubric.md    # 8-axis framework + academic citations
└── examples/
    ├── persona-english.md      # Example: English persona
    └── persona-korean.md       # Example: Korean persona
```

After running, the skill creates:
```
writer-persona/
├── persona.md                  # Your persona (gitignored — personal data)
└── reports/                    # Backtest reports (gitignored)
```

## Academic Foundations

| Concept | Source |
|---------|--------|
| Function word stylometry | Burrows' Delta (2002) |
| Multi-axis author identification | Writeprints, Abbasi & Chen (2008) |
| Formality benchmarks | GYAFC, Rao & Tetreault (2018) |
| LLM persona consistency | PersonaGym (EMNLP 2025) |
| Semantic preservation | BERTScore, Zhang et al. (2019) |
| Self-similarity baseline | TST 3-axis evaluation standard |

## License

MIT
