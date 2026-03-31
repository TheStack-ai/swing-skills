# Security Policy

## Scope

Swing is a collection of Markdown-based skill files that provide structured instructions to AI agents. The skills themselves do not execute code, access networks, or handle user data directly.

However, security concerns can arise from:
- Skills that instruct the AI to use tools (`WebSearch`, `WebFetch`, `Bash`, etc.) in unsafe ways
- Prompt injection vulnerabilities where skill instructions could be overridden
- Skills that inadvertently encourage the AI to expose sensitive information

## Reporting a vulnerability

If you discover a security issue, please report it responsibly:

**Email:** security@thestack.ai

Please include:
- Description of the vulnerability
- Which skill is affected
- Steps to reproduce
- Potential impact

We will acknowledge receipt within 48 hours and provide an initial assessment within 7 days.

## What qualifies as a security issue

- A skill instruction that could cause the AI to leak sensitive environment variables or credentials
- A skill that can be manipulated via prompt injection to bypass its own rules
- An `allowed-tools` declaration that grants unnecessary capabilities (e.g., granting `Bash` to a skill that only needs `Read`)
- A skill that instructs the AI to fetch/execute content from untrusted URLs without user consent

## What does NOT qualify

- AI output quality issues (use a bug report instead)
- The AI not following skill instructions perfectly (this is a calibration issue)
- General AI safety concerns not specific to Swing skills

## Supported versions

| Version | Supported |
|---------|-----------|
| Latest  | Yes       |
| Older   | No        |

We only address security issues in the latest release. Update to the latest version for security fixes.
