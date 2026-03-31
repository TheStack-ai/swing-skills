# Contributing to Swing

Thanks for your interest in contributing to Swing. This project builds cognitive firewalls for AI agents — structured skills that defend against specific reasoning failures.

## What you can contribute

### Improve existing skills
The highest-impact contributions improve the **rules**, **quality calibration examples**, or **process steps** of existing skills. Each skill in `skills/` has a `SKILL.md` file with:

- **Rules (Absolute)** — constraints the AI must follow
- **Process** — step-by-step pipeline
- **Output Format** — structured template
- **Quality Calibration** — BAD (what to avoid) and GOOD (what to aim for) examples
- **Integration Notes** — how the skill chains with others

Better calibration examples directly improve output quality across all platforms.

### Add new skills
New skills should address a **specific, named cognitive failure** that isn't covered by the existing six. Before building:

1. Identify the failure mode with a concrete example (like the SQLite/JWT examples in the README)
2. Open an issue describing the failure and proposed firewall
3. Get feedback before writing the full skill

### Fix documentation
README improvements, better install instructions, translations, and typo fixes are always welcome.

### Report quality issues
If a skill produces bad output on a specific platform, open a bug report with the prompt you used and the output you received. This helps calibrate the skill for edge cases.

## Skill file structure

Every skill lives in `skills/<skill-name>/SKILL.md` and must have:

```yaml
---
name: skill-name
description: One-line description with trigger keywords
argument-hint: "[what the user passes as argument]"
allowed-tools: Read, Grep, Glob, Bash
---
```

The body follows this structure:
1. **Title and introduction** — what cognitive failure this addresses
2. **Rules (Absolute)** — numbered, non-negotiable constraints
3. **Process** — sequential stages the AI must follow
4. **Output Format** — markdown template with placeholders
5. **Quality Calibration** — BAD and GOOD examples (mandatory for new skills)
6. **When to Use / When NOT to Use** — scope guidance
7. **Integration Notes** — how this skill chains with others

## Development workflow

1. Fork the repository
2. Create a branch: `git checkout -b feature/your-change`
3. Make your changes
4. Test with at least one AI platform (Claude Code, Cursor, Codex CLI, etc.)
5. Open a pull request with:
   - What you changed and why
   - How you tested it
   - Example prompt and output showing the improvement

## Testing skills

Since skills are Markdown instructions (not executable code), testing means:

1. Install the modified skill in your AI platform
2. Run representative prompts
3. Compare output quality against the BAD/GOOD calibration examples
4. Verify YAML frontmatter is valid
5. Check that the CI workflow passes (`validate-skills` and `markdown-lint` jobs)

## Style guidelines

- **Rules use absolute language.** "Never", "Always", "Must" — not "should", "consider", "try to".
- **Examples are specific.** Real technologies, real numbers, real scenarios — not generic placeholders.
- **BAD examples explain WHY they're bad.** Not just what's wrong, but the pattern to avoid.
- **GOOD examples are genuinely good.** They should pass the skill's own rules.
- **Keep skills focused.** One cognitive failure per skill. If your skill addresses two failures, split it.

## Code of Conduct

This project follows the [Contributor Covenant Code of Conduct](CODE_OF_CONDUCT.md). By participating, you agree to uphold it.

## Questions?

Open an issue with the `question` label, or reach out on [X/Twitter @thestack_ai](https://x.com/thestack_ai).
