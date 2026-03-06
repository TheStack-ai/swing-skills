---
name: cross-verified-research
description: Deep research with cross-verification and source tiering. Use when investigating technologies, comparing tools, fact-checking claims, evaluating architectures, or any task requiring verified information. Triggers on "조사해줘", "리서치", "research", "investigate", "fact-check", "비교 분석", "검증해줘".
argument-hint: "[topic or question to research]"
allowed-tools: WebSearch, WebFetch, Read, Grep, Glob, Bash, Agent
---

# Cross-Verified Research

Systematic research engine with anti-hallucination safeguards and source quality tiering.


## Rules (Absolute)

1. **Never fabricate sources.** No fake URLs, no invented papers, no hallucinated statistics.
2. **Confidence gate.** If confidence < 90% on a factual claim, do NOT present it as fact. State uncertainty explicitly.
3. **No speculation as fact.** Do not present unverified claims using hedging language as if they were findings. Banned patterns: "아마도", "~인 것 같습니다", "~로 보입니다", "~수도 있습니다", "probably", "I think", "seems like", "appears to be", "likely". If a claim is not verified, label it explicitly as **Unverified** or **Contested** — do not soften it with hedging.
4. **BLUF output.** Lead with conclusion, follow with evidence. Never bury the answer.
5. **Minimum effort.** At least 5 distinct search queries per research task. At least 5 verified sources in final output.
6. **Cross-verify.** Every key claim must appear in 2+ independent sources before presenting as fact.

## Pipeline

Execute these 4 stages sequentially. Do NOT skip stages.

### Stage 1: Deconstruct

Break the research question into atomic sub-questions.

```
Input: "Should we use Bun or Node.js for our backend?"
Decomposed:
  1. Runtime performance benchmarks (CPU, memory, startup)
  2. Ecosystem maturity (npm compatibility, native modules)
  3. Production stability (known issues, enterprise adoption)
  4. Developer experience (tooling, debugging, testing)
  5. Long-term viability (funding, community, roadmap)
```

- Identify what requires external verification vs. internal knowledge
- Flag any sub-question where confidence < 90%

### Stage 2: Search & Collect

For each sub-question requiring verification:

1. **Formulate diverse queries** — vary keywords, include year filters, try both English and Korean
2. **Use WebSearch** for broad discovery, **WebFetch** for specific page analysis
3. **Classify every source** by tier immediately (see Source Tiers below)
4. **Extract specific data points** — numbers, dates, versions, quotes with attribution
5. **Record contradictions** — when sources disagree, note both positions

Minimum search pattern:
```
Query 1: [topic] + "benchmark" or "comparison"
Query 2: [topic] + "production" or "enterprise"
Query 3: [topic] + [current year] + "review"
Query 4: [topic] + "issues" or "problems" or "limitations"
Query 5: [topic] + site:github.com (issues, discussions)
```

**Fallback when WebSearch is unavailable or returns no results:**
1. Use WebFetch to directly access known authoritative URLs (official docs, GitHub repos, Wikipedia)
2. Rely on internal knowledge but label all claims as **Unverified (no external search available)**
3. Ask the user to provide source URLs or documents for verification
4. Reduce the minimum source requirement but maintain cross-verification where possible

### Stage 3: Cross-Verify

For each key finding:
- Does it appear in 2+ independent Tier S/A sources? → **Verified**
- Does it appear in only 1 source? → **Unverified** (label it)
- Do sources contradict? → **Contested** (present both sides with tier labels)

Build a verification matrix:
```
| Claim | Source 1 (Tier) | Source 2 (Tier) | Status |
|-------|----------------|----------------|--------|
| Bun 3x faster startup | benchmarks.dev (A) | bun.sh/blog (B) | Verified (note: Bun's own blog = biased) |
```

### Stage 4: Synthesize

Produce the final report in BLUF format.

## Output Format

```markdown
## Research: [Topic]

### Conclusion (BLUF)
[1-3 sentence definitive answer or recommendation]

### Key Findings
[Numbered findings, each with inline source tier labels]

1. **[Finding]** — [evidence summary]
   Sources: 🏛️ [source1], 🛡️ [source2]

2. **[Finding]** — [evidence summary]
   Sources: 🛡️ [source1], 🛡️ [source2]

### Contested / Uncertain
[Any claims that couldn't be cross-verified or where sources conflict]
- ⚠️ [claim] — Source A says X, Source B says Y

### Verification Matrix
| Claim | Sources | Tier | Status |
|-------|---------|------|--------|
| ... | ... | ... | Verified/Unverified/Contested |

### Sources
[All sources, grouped by tier]

#### 🏛️ Tier S — Academic & Primary Research
- [Title](URL) — Journal/Org (Year)

#### 🛡️ Tier A — Trusted Official
- [Title](URL) — Source (Year)

#### ⚠️ Tier B — Community / Caution
- [Title](URL) — Platform (Year)

#### Tier C — General
- [Title](URL)
```

## Source Tiers

Classify every source on discovery.

| Tier | Label | Trust Level | Examples |
|------|-------|-------------|----------|
| S | 🏛️ | Academic, peer-reviewed, primary research, official specs | Google Scholar, arXiv, PubMed, W3C/IETF RFCs, language specs (ECMAScript, PEPs) |
| A | 🛡️ | Government, .edu, major press, official docs | .gov/.edu, Reuters/AP/BBC, official framework docs, company engineering blogs (Google AI, Netflix Tech) |
| B | ⚠️ | Social media, forums, personal blogs, wikis — flag to user | Twitter/X, Reddit, StackOverflow, Medium, dev.to, Wikipedia, 나무위키 |
| C | (none) | General websites not fitting above categories | Corporate marketing, press releases, SEO content, news aggregators |

### Tier Classification Rules

- **Company's own content about their product:**
  - Official docs → Tier A
  - Feature announcements → Tier A (existence), Tier B (performance claims)
  - Marketing pages → Tier C
- **GitHub:**
  - Official repos (e.g., facebook/react) → Tier A
  - Issues/Discussions with reproduction → Tier A (for bug existence)
  - Random user repos → Tier B
- **Benchmarks:**
  - Independent, reproducible, methodology disclosed → Tier S
  - Official by neutral party → Tier A
  - Vendor's own benchmarks → Tier B (note bias)
- **StackOverflow:** Accepted answers with high votes = borderline Tier A; non-accepted = Tier B
- **Tier B sources must never be cited alone** — corroborate with Tier S or A

## When to Use

- Technology evaluation or comparison
- Fact-checking specific claims
- Architecture decision research
- Market/competitor analysis
- "Is X true?" verification tasks
- Any question where accuracy matters more than speed

## When NOT to Use

- Creative writing or brainstorming (use `creativity-sampler`)
- Code implementation (use `search-first` for library discovery)
- Simple questions answerable from internal knowledge with high confidence
- Opinion-based questions with no verifiable answer

## Integration Notes

- **With brainstorming:** Can be invoked during brainstorming's "Explore context" phase for fact-based inputs
- **With search-first:** search-first finds tools/libraries to USE; this skill VERIFIES factual claims. Different purposes.
- **With adversarial-review:** Research findings can feed into adversarial review for stress-testing conclusions
