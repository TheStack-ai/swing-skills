<p align="center">
  <img src="assets/swing-icon.svg" width="80" alt="Swing logo">
</p>

<h1 align="center">Swing — AI Agent Guardrails for Claude Code</h1>

<p align="center">
  <a href="https://github.com/whynowlab/swing-skills/stargazers"><img src="https://img.shields.io/github/stars/whynowlab/swing-skills?style=flat&color=58a6ff" alt="Stars"></a>
  <a href="https://github.com/whynowlab/swing-skills/releases"><img src="https://img.shields.io/github/v/tag/whynowlab/swing-skills?label=version&color=3fb950" alt="Version"></a>
  <a href="https://github.com/whynowlab/swing-skills/blob/main/LICENSE"><img src="https://img.shields.io/github/license/whynowlab/swing-skills?color=8b949e" alt="License"></a>
  <a href="https://github.com/whynowlab/swing-skills/commits/main"><img src="https://img.shields.io/github/commit-activity/m/whynowlab/swing-skills?color=58a6ff" alt="Commits"></a>
  <a href="https://x.com/thestack_ai"><img src="https://img.shields.io/badge/follow-@thestack__ai-1DA1F2?logo=x&logoColor=white" alt="Twitter"></a>
  <br>
  <img src="https://img.shields.io/badge/skills-6_firewalls-58a6ff" alt="6 Firewalls">
  <img src="https://img.shields.io/badge/Claude_Code-compatible-bc8cff?logo=anthropic&logoColor=white" alt="Claude Code">
  <img src="https://img.shields.io/badge/Codex_CLI-compatible-3fb950?logo=openai&logoColor=white" alt="Codex CLI">
  <a href="https://github.com/whynowlab/swing-skills/pulls"><img src="https://img.shields.io/badge/PRs-welcome-3fb950" alt="PRs Welcome"></a>
</p>

<p align="right">
  <a href="README.md">English</a> · <a href="README.ko.md">한국어 (Korean)</a>
</p>

<p align="center">
  <img src="assets/swing-hero-terminal.png" width="800" alt="Swing terminal installation">
</p>

**Your AI is not swinging at all.**

It picks the safe default, agrees when it should push back, cites no sources, hides its reasoning, and assumes everything will work. Six cognitive failures. Six firewalls. Make it swing.

```
npx skills add whynowlab/swing-skills --all
```

> Built for [Claude Code](https://claude.com/claude-code) | Compatible with AI agents via [Agent Skills](https://agentskills.io) open standard
> Created by [@thestack_ai](https://github.com/whynowlab)

---

## See the Difference

### Before: Default AI Research

> **You:** Is SQLite viable for an app with 1000 concurrent users?
>
> **AI:** No, SQLite is not suitable for high-concurrency applications. Use PostgreSQL or MySQL instead for production workloads.

### After: With `swing-research`

> **You:** /swing-research Is SQLite viable for an app with 1000 concurrent users?
>
> **AI:** **Yes, with caveats.** 1000 concurrent users ≠ 1000 concurrent writes. Typical web apps generate ~30 concurrent write transactions at this scale. SQLite handles ~120 write transactions/sec in WAL mode (verified: sqlite.org benchmarks + Expensify production case study serving 10M+ users on SQLite). The real constraint is write contention above ~100 concurrent writers, not user count.
>
> Sources: 🏛️ sqlite.org WAL documentation, 🛡️ Expensify engineering blog (2024), ⚠️ HN discussion (corroborated)

**The baseline gave a wrong answer confidently. The skill forced source verification and reached the opposite — correct — conclusion.**

### Before: Default AI Code Review

> **You:** Review this JWT auth module
>
> **AI:** The code looks well-structured. Consider adding more comments and using TypeScript for better type safety. Variable naming could be improved in some places.

### After: With `swing-review`

> **You:** /swing-review Review this JWT auth module
>
> **AI:**
> **Steel-man:** JWT-based stateless auth was chosen for horizontal scaling. The 15-min access + 7-day refresh split balances security and UX.
>
> 🔴 **Critical: No refresh token rotation.** A stolen token grants persistent 7-day access with no detection. Fix: implement token family chain rotation.
>
> 🟠 **Major: Rate limiting uses in-memory Map().** Resets on restart, bypassed in multi-instance deploys. Fix: move to Redis with TTL.
>
> **Verdict: PASS WITH CONDITIONS** — must fix token rotation before deploy.

**The baseline found nothing. The skill found a Critical security vulnerability.**

---

## Six Failures, Six Firewalls

| Cognitive Failure | What Happens | Firewall | What It Forces |
|:---|:---|:---|:---|
| **Premature closure** | AI rushes to execute on ambiguous requests | `swing-clarify` | Structured clarification before execution begins |
| **Hallucination** | AI states claims without verification | `swing-research` | Source-traced, cross-verified claims with S/A/B/C tier grading |
| **Anchoring bias** | AI locks onto the first "obvious" answer | `swing-options` | 5 probability-weighted options including unconventional alternatives |
| **Confirmation bias** | AI agrees with you instead of challenging | `swing-review` | Steel-man then 3-vector attack. "Looks good" is structurally banned |
| **Black-box reasoning** | AI gives conclusions without showing why | `swing-trace` | Assumption inventory and weakest-link analysis |
| **Optimism bias** | AI assumes the plan will work | `swing-mortem` | Assumes failure, identifies 5 specific failure scenarios with circuit breakers |

---

## Skills

### swing-clarify

Prevents premature execution on ambiguous requests.

```
Request → 5W1H Decomposition → Ambiguity Score → Clarify or Proceed
```

- Breaks every request into 6 dimensions: What, Who, Where, When, Why, How
- Scores ambiguity (0-6) — clear requests get a green light, ambiguous ones get up to 3 questions
- Every question includes multiple choice options and a stated default assumption
- Designed to run FIRST — before any other skill in the chain

```
Try: /swing-clarify Build me an auth system
```

### swing-research

4-stage verified research pipeline with anti-hallucination safeguards.

```
Deconstruct → Search & Collect → Cross-Verify → Synthesize
```

- Every claim must be traced to a specific, citable source — or labeled `Unverified`
- Source tiering: S (academic/specs), A (official docs), B (community — flagged), C (general)
- Cross-verification: key claims require 2+ *independent* sources (same-origin rewrites don't count)
- Adaptive depth: narrow questions get 2-3 queries, broad landscape gets 8+

```
Try: /swing-research Is gRPC better than REST for mobile backends?
```

### swing-options

Probability-weighted option generator that breaks anchoring bias.

- 5 options by default, each assigned a typicality zone (Conventional → Wild card)
- At least 1 option from the unconventional or wild card zone — ideas your AI normally suppresses
- **Hidden Assumptions** section exposes *why* the obvious answer seems obvious
- Decision matrix with criteria derived from your stated constraints + 1 hidden criterion you didn't consider

```
Try: /swing-options Which database for a real-time leaderboard?
```

### swing-review

Structured Devil's Advocate that finds real problems, not nitpicks.

- **Steel-man first**: articulates the strongest case FOR the current approach before attacking
- **3 independent vectors**: Logical Soundness / Edge Case Assault / Structural Integrity
- Severity classification: Critical (must fix) / Major (should fix) / Minor / Note
- Every Critical and Major finding includes a counter-proposal with trade-off analysis
- Explicit verdict thresholds: any unmitigated Critical = FAIL

```
Try: /swing-review We chose Kubernetes for our 3-person startup
```

### swing-trace

Makes AI reasoning visible, auditable, and decomposable.

- **Assumption inventory**: every assumption numbered, rated for criticality and verifiability
- **Decision tree**: at each reasoning fork, what alternative was considered and why it was rejected
- **Confidence decomposition**: overall confidence broken into sub-components with justifications
- **Weakest link**: which single assumption, if wrong, would most change the conclusion
- **Alternative conclusion**: "If [weakest assumption] is wrong, then the answer changes to [X]"

```
Try: /swing-trace Why do you recommend microservices for this project?
```

### swing-mortem

Prospective failure analysis — assumes your plan failed and works backward.

- "It is 6 months from now. This failed completely. What went wrong?"
- 5 failure scenarios across categories: Technical / Organizational / External / Temporal / Assumption
- Likelihood x Impact matrix → focus on top 3 risks
- **Leading indicators**: measurable early-warning signals for each top risk
- **Circuit breakers**: specific trigger conditions for when to stop and pivot

```
Try: /swing-mortem We're migrating our monolith to microservices over Q3
```

---

## Which Skill Should I Use?

### Daily Use (Flagship)

| Your situation | Skill | Example |
|:---|:---|:---|
| Request is vague or has implicit assumptions | `swing-clarify` | "Build me an auth system" → clarifies SSO? RBAC? OAuth? |
| Researching a technology or verifying a claim | `swing-research` | "Is gRPC better than REST for mobile?" |
| Reviewing code, architecture, or a decision | `swing-review` | "We chose Kubernetes for our 3-person startup" |

### High-Stakes Moments

| Your situation | Skill | Example |
|:---|:---|:---|
| Choosing between options | `swing-options` | "Which database for real-time leaderboard?" |
| Planning a project, want to de-risk | `swing-mortem` | "We're migrating to microservices in Q3" |

### Deep Analysis

| Your situation | Skill | Example |
|:---|:---|:---|
| Want to see WHY the AI recommends something | `swing-trace` | "Why microservices for this project?" |
| Need full audit trail | `swing-trace --full` | Architecture decision records |

### Recommended Chains

- **Any Task**: `swing-clarify` → [any other skill] (always clarify first)
- **Tech Decision**: `swing-clarify` → `swing-options` → `swing-research` → `swing-review`
- **Architecture Review**: `swing-trace` → `swing-review`
- **Project Kickoff**: `swing-mortem` → `swing-options` (for mitigations)

---

## How They Work Together

```
Your AI's default reasoning:

  [Question] ──→ [First plausible answer] ──→ [Ship it]


With Swing:

  [Request]
       │
       ├──→ swing-clarify ──→ "What EXACTLY do you need?"  (prevents premature closure)
       │
       ├──→ swing-options ──→ "What are ALL the options?"  (breaks anchoring)
       │
       ├──→ swing-research ──→ "Is this actually true?"  (blocks hallucination)
       │
       ├──→ swing-trace ──→ "WHY do I believe this?"  (exposes assumptions)
       │
       ├──→ swing-review ──→ "What's WRONG with this?"  (kills confirmation bias)
       │
       └──→ swing-mortem ──→ "HOW will this FAIL?"  (counters optimism bias)
```

Each firewall is independent — use one, or chain them.

---

## Why a Skill?

You could paste these instructions into CLAUDE.md or a system prompt. But:

1. **Context efficiency** — Skills load only when triggered (~100 tokens to scan, full instructions only when invoked). A system prompt pays the token cost every message.
2. **Composability** — Skills chain: `swing-clarify` → `swing-research` → `swing-review`. A monolithic prompt can't be selectively invoked.
3. **Portability** — `npx skills add` works across Claude Code, Cursor, Copilot. A system prompt is locked to one tool.
4. **Community** — Skills can be shared, forked, and improved. A prompt in your dotfiles helps only you.

---

## Install

### Quick Install (npx)

```bash
npx skills add whynowlab/swing-skills --all
```

### Individual Skills

```bash
npx skills add whynowlab/swing-skills@swing-clarify
npx skills add whynowlab/swing-skills@swing-research
npx skills add whynowlab/swing-skills@swing-review
npx skills add whynowlab/swing-skills@swing-options
npx skills add whynowlab/swing-skills@swing-trace
npx skills add whynowlab/swing-skills@swing-mortem
```

### Manual

```bash
git clone https://github.com/whynowlab/swing-skills.git
cp -r swing-skills/skills/* ~/.claude/skills/
```

### Per-Project

```bash
cp -r swing-skills/skills/swing-review .claude/skills/
```

---

## Compatibility

| Platform | Status |
|:---------|:-------|
| Claude Code | Fully supported |
| Cursor | Compatible (Agent Skills standard) |
| GitHub Copilot | Compatible (Agent Skills standard) |
| Codex CLI | Compatible (Agent Skills standard) |

---

## License

MIT License. See [LICENSE](LICENSE).

---

**Swing** by [@thestack_ai](https://github.com/whynowlab) — 6 cognitive firewalls for your AI.