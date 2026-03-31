---
name: writer-persona
description: "Extract your writing persona from real messages, measure its accuracy, and write in your voice — powered by an 8-axis linguistic evaluation framework."
---

# Writer Persona

Extract your writing persona from real conversations. Measure how accurately AI can mimic you. Then let it write as you.

## Usage

```
/writer-persona                        # Show persona status (score, last backtest)
/writer-persona --bootstrap            # Extract persona from 100+ real messages
/writer-persona --backtest             # Measure accuracy (default: last 24h)
/writer-persona --backtest --days 7    # Measure over last 7 days
/writer-persona --calibrate            # Manual tuning with your feedback
/writer-persona --write "context"      # Write as you in a given situation
/writer-persona --write                # Interactive: ask for context, then write
```

## Execution

### Default (no args): Status

1. Read persona file at `persona.md` in this skill's directory
2. If exists, display:
   ```
   Writer Persona: {user_name}
   Score: {last_score}/100 (ceiling: {ceiling_score})
   Messages analyzed: {sample_count}
   Last updated: {last_updated}
   ```
3. If no persona file: "No persona yet. Run `/writer-persona --bootstrap` to get started."

---

### Phase 0: Setup & Persona Load

1. Read persona file at `persona.md` in this skill's directory (or path configured in frontmatter)
   - If the file doesn't exist, copy `persona.template.md` from this skill's directory
2. Check frontmatter:
   - `sample_count: 0` or file missing → auto-enter Phase B (bootstrap)
   - `--bootstrap` flag → force Phase B (overwrites existing persona)
3. If `user_name` or `data_source` is empty → enter **First Run Setup** (below)

#### First Run Setup (interactive)

Ask the user these questions using AskUserQuestion, one at a time:

1. **"What name should I use for the persona?"**
   - Store as `user_name` in frontmatter

2. **"What language do you primarily write in?"**
   - Examples: English, Korean, Japanese, Spanish...
   - Store as `language` in frontmatter

3. **"How should I collect your messages? Describe your data source."**
   - Explain: "I need access to 100+ of your real messages for bootstrap, and recent conversations for backtesting. This could be Slack (via MCP), email, Discord, Teams, a text file, or anything your MCP tools can access."
   - Examples to offer:
     - "Slack — I have mcp-slack tools available"
     - "I'll paste messages manually"
     - "I have a file with exported messages"
     - "Email — I have an email MCP"
     - Other (describe)
   - Store the description as `data_source` in frontmatter

4. **"What tool or method should I use to search/collect your messages?"**
   - Ask for specifics based on their answer:
     - For Slack: "What's your Slack username or user ID? Which MCP tool should I use to search messages?"
     - For manual: "Please provide a file path, or I'll ask you to paste messages during bootstrap."
     - For other: "Describe the MCP tool name and how to query it, or how you'll provide messages."
   - Store as `data_source_config` in frontmatter (freeform object)

5. **"Are there any situation categories specific to your work?"**
   - Default categories: Technical Q&A, Work coordination, Feedback/review, Casual chat, Formal/leadership
   - User can customize (e.g., add "Client communication", remove "Leadership")
   - Store in frontmatter as `situation_categories`

Save the configured persona file and proceed to Phase B.

---

### Phase B: Bootstrap (Initial Construction)

**Goal**: Analyze 100+ real messages to build the persona definition.

Use `Task` tool with `model: "sonnet"`.

**Include in the agent prompt**:

- The full `persona.template.md` content
- The user's `data_source_config` (how to collect messages)
- The `language` setting
- Instructions below:

#### B-1. Message Collection

Collect **100+ messages** written by the user, using whatever data source they configured.

- Aim for the **last 30 days** to capture current style
- Ensure **diversity**: different channels/contexts, different recipients, different times of day
- If using an MCP tool, paginate as needed to reach 100+
- If manual input, ask the user to provide messages (or read from a specified file)

**Message format** (normalize all sources to this):
```
CONTEXT: [channel/thread/email subject]
TO: [recipient(s)]
DATE: [ISO date]
---
[message text]
===
```

#### B-2. Message Classification

Classify each message into the configured situation categories.
If a message doesn't fit any category, assign "Other" and note the pattern — this may indicate a missing category.

#### B-3. 8-Axis Analysis

For each axis, analyze patterns across all collected messages. See `references/evaluation-rubric.md` for axis definitions.

Analyze in the user's configured `language`. Adapt axis-specific features to the language:
- **Korean**: Track 합쇼체/해요체/해체, ㅋㅎ patterns, particle omission
- **English**: Track contractions, hedging phrases, slang density, register markers
- **Japanese**: Track keigo levels, sentence-final particles, kanji/kana ratio
- **Other languages**: Identify the equivalent formality/register markers

Produce:
1. **Lexical Choice**: Top 30 characteristic words/phrases, avoided expressions, abbreviation patterns, foreign word ratio
2. **Function Words**: Particle/preposition preferences, conjunction patterns, high-frequency function words (Top 20)
3. **Sentence Structure**: Average sentence length (chars ± SD), word/sentence counts, simple:complex ratio
4. **Formality / Register**: Distribution across formality levels (% breakdown), context-specific switching rules
5. **Tone / Sentiment**: Base tone keywords, positive/negative/neutral ratio, emotion expression frequency
6. **Formatting**: Emoji usage rate, favorites, laugh markers, punctuation density, line break patterns
7. **Discourse Patterns**: Greeting/closing style, question rate, response structure, context referencing method

#### B-4. Situational Tone Map

For each situation category, document: typical tone, register level, and one example message.

#### B-5. Self-Similarity Ceiling

1. Randomly select **20 message pairs** from the same situation category
2. Score each pair on 8 axes using the Phase 3 scoring method ("do these look like the same person?")
3. Average of 20 pairs = **ceiling score** (this becomes the "100 point" baseline)
4. Record standard deviation
5. Typical range: 65-85 (people vary naturally)

#### B-6. Golden Set

Select **2 representative messages per situation category** = ~10 example messages.
Choose messages that best exemplify the user's style in each context.

#### B-7. Save Persona

Write all analysis results to the persona file following the template structure.
Update frontmatter: `sample_count`, `last_updated`, `ceiling_score`, `ceiling_stddev`.
Add to Update History: "Initial bootstrap (N messages, last 30 days)"

**Completion message**: "Writer Persona ready! Analyzed N messages. Self-similarity ceiling: XX. Try `/writer-persona --backtest` to measure accuracy, or `/writer-persona --write` to write in your voice."

---

### Phase 1: Mention/Conversation Collection

**Goal**: Find recent conversations where the user responded, and extract context + actual response.

Use `Task` tool with `model: "sonnet"`.

**Include in the agent prompt**:

- The user's `data_source_config`
- Period: `--days N` (default: 1)
- Instructions:

1. **Search for conversations** where the user was mentioned or participated, within the specified period
   - Use whatever data source the user configured
   - Look for messages directed at the user, threads the user replied to, or conversations the user participated in

2. **For each conversation**, extract:
   - `situation_context`: All messages **before** the user's response (the setup)
   - `actual_reply`: The user's actual response
   - `channel_name`: Where the conversation happened (channel, email thread, DM, etc.)
   - `participants`: Who else was in the conversation
   - `timestamp`: When the user responded

3. **Filter out**:
   - Responses that are 1 word or less
   - Emoji-only reactions
   - Minimal acknowledgments ("ok", "sure", "got it", "ㅇㅇ", "ㅋㅋ")

4. **Return**: Array of `{situation_context, actual_reply, channel_name, participants, timestamp}`

**Minimum**: 5 cases required. If fewer, suggest extending the period (e.g., "Only N cases in 24h. Try `--backtest --days 3`.")

---

### Phase 2: Draft Generation (Parallel, Information-Isolated)

Process up to **5 cases in parallel** using separate Task agents.

Each agent (`Task` tool, `model: "sonnet"`):

**CRITICAL**: The draft generation agent must **NEVER** see the `actual_reply`. This is the core fairness guarantee of the backtest. Only provide `situation_context`.

**Prompt for each agent**:

```
You are writing as "{user_name}" based on the persona definition below.

## Persona Definition
{full persona file content}

## Situation
Channel/Context: {channel_name}
Participants: {participants}
Conversation so far:
{situation_context}

## Instructions
Write the response that {user_name} would send in this situation.
- Follow the persona's vocabulary, tone, register, and formatting patterns exactly
- Match the situational tone map for this type of conversation
- Infer what {user_name} would say based on their role and expertise
- Write in {language}
- Output ONE message only (in the platform's message format)
- Output the message only. No explanations, labels, or meta-commentary.
```

---

### Phase 3: Scoring

Score each `{draft, actual_reply}` pair. **This phase uses the default model (opus-level recommended)** — complex judgment required.

#### 3-1. Structural Analysis (Direct Calculation)

Compare the two texts numerically:
- Sentence count difference
- Average sentence length difference (chars)
- Formality/register marker matching
- Emoji count difference
- Laugh marker count difference (lol/haha/ㅋㅋ/www etc.)
- Line break count difference
- Total character count difference (%)

#### 3-2. LLM-as-Judge (8-Axis Scoring)

**Scoring prompt**:

```
You are an expert in writing style analysis. Evaluate whether two texts appear to be written by the same person, across 8 linguistic axes.

## Self-Similarity Ceiling
Even the same person varies across messages.
Self-similarity ceiling for this user: {ceiling_score} points.
This ceiling IS the 100-point baseline. Score relative to it:
"Within the range of natural variation for this person" = full marks on that axis.

## Actual Response (ground truth)
{actual_reply}

## AI Draft
{draft}

## Situation Context
{situation_context}

## Structural Analysis
{Phase 3-1 metrics}

## Evaluation Language
Evaluate in {language}. Use language-appropriate criteria for each axis.

## Score each axis (0-100, normalized to ceiling)

Output JSON:
{
  "lexical": { "score": N, "note": "one-line rationale" },
  "function_words": { "score": N, "note": "..." },
  "sentence_structure": { "score": N, "note": "..." },
  "formality": { "score": N, "note": "..." },
  "tone": { "score": N, "note": "..." },
  "formatting": { "score": N, "note": "..." },
  "semantic": { "score": N, "note": "..." },
  "discourse": { "score": N, "note": "..." },
  "weighted_total": N,
  "top_gaps": ["axis names with lowest scores, 1-2"],
  "persona_fix": "How to modify the persona definition to close this gap"
}

Weights: semantic 25%, tone 20%, formality 15%, discourse 15%, lexical 10%, function_words 5%, sentence_structure 5%, formatting 5%
```

---

### Phase 4: Report Generation

Aggregate all case scores into a report.

**Output path**: `{skill_directory}/reports/YYYY-MM-DD-backtest.md`
(Create the `reports/` directory if it doesn't exist)

**Report format**:

```markdown
---
tags: [writer-persona, backtest]
score: {weighted_total_average}
cases: {N}
period: {description}
---

# Writer Persona Backtest — {YYYY-MM-DD}

> Overall: **{XX}/100** | Cases: {N} | Period: last {N} days | Ceiling: {ceiling_score}

## Summary

| Axis | Score | Weight | Note |
|------|-------|--------|------|
| Semantic Fidelity (25%) | {XX} | | {note} |
| Tone / Sentiment (20%) | {XX} | | |
| Formality / Register (15%) | {XX} | | |
| Discourse Patterns (15%) | {XX} | | |
| Lexical Choice (10%) | {XX} | | |
| Function Words (5%) | {XX} | | |
| Sentence Structure (5%) | {XX} | | |
| Formatting (5%) | {XX} | | |
| **Weighted Total** | **{XX}** | | |

## Key Findings

{2-3 bullet points summarizing the most important patterns}

## Case Details

### Case {N}: {channel_name} — {one-line summary}

**Situation**: {1-2 line context}
**Actual response**:
> {actual text}

**AI draft**:
> {draft text}

**Score**: {XX}/100
**Main gap**: {one line}

---

{repeat for each case}

## Persona Corrections {only if score < 80}

### Modified
- **{axis name}**: {before} → {after}
  - Evidence: {N} cases flagged this pattern

### Verify Next Run
- {specific thing to check}

---
_Generated by [writer-persona](https://github.com/cosmos-makers/writer-persona)_
```

---

### Phase 5: Auto-Correction (Conditional)

**Trigger**: Weighted total score < 80

1. Collect all `persona_fix` suggestions from Phase 3
2. **Find common patterns**: Same axis flagged in 2+ cases
3. **Modify at most 2 axes** (highest gap first)
4. Edit the persona file's corresponding sections
5. Update frontmatter: `last_updated`, `last_score`
6. Add to Update History: what changed, why, which cases provided evidence

**Overfitting prevention**:
- Maximum 2 axes per run (prevents overcorrection)
- If the same axis has been corrected 3 consecutive times in the same direction WITHOUT score improvement → stop auto-correcting that axis
  - Mark in report: "Auto-correction paused for {axis} — use `--calibrate` for manual tuning"
- Record before/after values for every change (enables rollback)

---

### --calibrate Mode

Manual persona tuning with user feedback.

1. Load the most recent backtest report from `{skill_directory}/reports/`
2. For each case, show side by side:
   - The situation context
   - The actual response
   - The AI draft
3. Ask the user:
   - "Which sounds more like you — the actual or the draft?"
   - "What specifically feels off about the draft?"
4. Apply feedback to the persona file
5. Can unblock axes that auto-correction paused

---

### --write Mode

Write a message in the user's voice.

1. **Load persona** (refuse if `sample_count: 0` → guide to `--bootstrap`)
2. **Get context** from the user:
   - If argument provided (`--write "reply to John's email about the deadline"`): use that as context
   - If no argument: ask interactively:
     - "Who are you writing to?"
     - "What's the situation/context?"
     - "What do you want to communicate?"
     - "What platform is this for?" (email, Slack, DM, etc.)
3. **Determine situation category** from the context → select matching register/tone from persona
4. **Generate draft** using the same prompt template as Phase 2:
   - Full persona definition
   - Situation context from user input
   - Matching situational tone map entry
5. **Output the draft** to the user
6. **Ask**: "Want me to adjust anything? (tone, length, formality, content)"
   - If yes: regenerate with the adjustment
   - If no: done

The `--write` mode is the payoff of all the extraction and calibration work. The better the persona score, the better the drafts.

---

## Important Rules

### Do

- Run message collection and draft generation as **separate agents** (information isolation — draft agents never see actual responses)
- Always measure and apply the self-similarity ceiling
- Exclude trivial responses (1-word, emoji-only, minimal acks)
- Record all persona changes with diffs and evidence
- Use the strongest available model for scoring (complex multi-axis judgment)
- Adapt all analysis to the user's configured language

### Don't

- Start backtesting without bootstrap (`sample_count: 0` → refuse, guide to `--bootstrap`)
- Modify 3+ axes simultaneously
- Judge scores without ceiling context (raw scores are meaningless without the baseline)
- Expose actual responses outside the local environment
- Show actual responses to draft-generation agents (breaks the fairness guarantee)

## Academic Foundations

See `references/evaluation-rubric.md` for the full framework and citations:
- **Burrows' Delta** (2002) — function word stylometry
- **Writeprints** (Abbasi & Chen, 2008) — multi-axis author identification
- **GYAFC** (Rao & Tetreault, 2018) — formality benchmarks
- **PersonaGym** (EMNLP 2025) — LLM persona consistency evaluation
- **BERTScore** (Zhang et al., 2019) — semantic preservation
- **TST 3-axis** — style transfer evaluation standard
