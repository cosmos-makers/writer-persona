# 8-Axis Evaluation Rubric

A linguistically grounded framework for measuring how accurately an AI persona mimics a person's writing style. Each axis targets a distinct dimension of authorial voice, informed by computational stylometry research.

## Axes

### 1. Lexical Choice (Weight: 10%)
**What it measures**: Vocabulary selection — characteristic words, avoided expressions, abbreviation patterns, foreign/loanword ratio.

**Why it matters**: Word choice is the most immediately recognizable marker of individual style. People develop idiosyncratic vocabularies that are surprisingly consistent across contexts.

**How to score**: Compare the draft's vocabulary against the persona's documented characteristic expressions. Check for presence of signature phrases and absence of avoided terms.

### 2. Function Words (Weight: 5%)
**What it measures**: Distribution of grammatical particles, conjunctions, filler words, and high-frequency structural words.

**Why it matters**: Function words are largely unconscious choices, making them robust authorship signals. Burrows' Delta (2002) demonstrated that function word frequencies alone can identify authors with high accuracy.

**How to score**: Compare particle preferences, conjunction patterns, and omission habits between draft and actual message.

### 3. Sentence Structure (Weight: 5%)
**What it measures**: Average sentence length (characters ± standard deviation), simple vs. complex sentence ratio, sentences per message.

**Why it matters**: Syntactic complexity is a stable individual trait. Some people consistently write short punchy sentences; others embed multiple clauses.

**How to score**: Calculate structural metrics for both texts and compare differences.

### 4. Formality / Register (Weight: 15%)
**What it measures**: The level of formality and how it shifts across contexts — formal/informal markers, honorific levels (in languages with them), register switching rules.

**Why it matters**: Register is context-dependent but rule-governed. A person may use formal language with superiors and casual language with peers, but the switching rules are consistent and personal.

**How to score**: Check whether the draft uses the correct register for the given situation, matching the persona's documented situational register map.

### 5. Tone / Sentiment (Weight: 20%)
**What it measures**: Emotional coloring — base tone (warm, direct, reserved, etc.), positive/negative/neutral ratio, how emotions are expressed (exclamations, hedging, humor).

**Why it matters**: Tone is perhaps the most noticed aspect of "sounding like someone." Getting tone wrong is immediately jarring, even if vocabulary and structure are correct.

**How to score**: Assess whether the draft's emotional temperature matches the actual response — enthusiasm level, humor usage, directness, empathy markers.

### 6. Formatting (Weight: 5%)
**What it measures**: Punctuation habits, emoji usage rate and selection, line break patterns, use of bold/links/code blocks, laugh markers (e.g., "haha", "lol", "ㅋㅋ").

**Why it matters**: Formatting is a highly personal digital fingerprint. Emoji selection, punctuation density, and message structure (one long message vs. multiple short ones) are distinctive markers.

**How to score**: Compare emoji counts, punctuation patterns, line breaks, and formatting choices between draft and actual.

### 7. Semantic Fidelity (Weight: 25%)
**What it measures**: Whether the draft says the right *thing* — information accuracy, topic coverage, level of specificity, technical detail inclusion.

**Why it matters**: Style without substance is hollow. The AI must not only sound like the person but also respond with the kind of information and at the level of detail that person would provide.

**How to score**: Assess whether the draft addresses the same points as the actual response, with comparable specificity and accuracy.

### 8. Discourse Patterns (Weight: 15%)
**What it measures**: Conversational structure — greeting/closing style, response organization (conclusion-first vs. background-first), context referencing method, question frequency, message sequencing patterns.

**Why it matters**: How someone structures their responses is a deep behavioral pattern. Some people always lead with the answer; others build context first. These patterns are consistent and identifiable.

**How to score**: Compare the organizational structure, opening/closing patterns, and information flow between draft and actual.

## Weight Rationale

| Axis | Weight | Rationale |
|------|--------|-----------|
| Semantic Fidelity | 25% | Wrong content is worse than wrong style |
| Tone / Sentiment | 20% | Most noticed by readers; biggest "uncanny valley" risk |
| Formality / Register | 15% | Wrong register is socially jarring |
| Discourse Patterns | 15% | Deep behavioral signature; hard to fake |
| Lexical Choice | 10% | Recognizable but partially captured by other axes |
| Function Words | 5% | Subtle signal; primarily useful in aggregate |
| Sentence Structure | 5% | Structural variation is high even within one person |
| Formatting | 5% | Important but platform-dependent |

## Self-Similarity Ceiling

A key innovation: even the same person doesn't write identically every time. The **self-similarity ceiling** measures natural variation by scoring pairs of real messages from the same person in similar situations.

**Procedure**:
1. Select 20 message pairs from the same situation category
2. Score each pair on the 8 axes as if comparing "draft vs actual"
3. The average score = ceiling (this becomes the "100 point" baseline)
4. Typical ceilings: 70-80 (meaning a perfect AI should score ~70-80, not 100)

**Why this matters**: Without a ceiling, a score of 70 might look mediocre. With a ceiling of 75, that same 70 is actually excellent — within natural human variation.

## Academic Foundations

| Concept | Source | Contribution |
|---------|--------|-------------|
| Function word distance | Burrows' Delta (2002) | Stylometry de facto standard; showed function words alone identify authors |
| Multi-axis feature sets | Writeprints (Abbasi & Chen, 2008) | 100+ features achieved 94% author identification |
| Formality classification | GYAFC (Rao & Tetreault, 2018) | Grammarly corpus for formal/informal benchmarking |
| LLM persona consistency | PersonaGym (EMNLP 2025) | 5-point rubric + LLM cross-evaluation for persona fidelity |
| Semantic preservation | BERTScore (Zhang et al., 2019) | Style-independent content evaluation |
| Self-similarity baseline | TST 3-axis evaluation | Style + Content + Fluency standard from text style transfer |
