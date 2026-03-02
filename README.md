# Miscellaneous Claude Code Skills

Standalone skills for [Claude Code](https://docs.anthropic.com/en/docs/claude-code) — code review and decision analysis.

## Skills

### grill

Adversarial 2-phase code review built on community sources. A Comprehensive Reviewer agent finds issues across 7 categories, then a GPTLens Critic agent validates each finding to eliminate false positives. Claude Code consolidated the sources; Joon Chung designed the 2-agent architecture and review workflow.

- **5C audit findings** (Condition, Criteria, Cause, Effect, Recommendation — IIA Standards 2410/2420)
- **OWASP risk scoring** (Likelihood x Impact, 0-9 scale)
- **GPTLens Auditor/Critic pattern** (Hu et al., IEEE TPS 2023)
- Built on: [trailofbits/claude-code-config](https://github.com/trailofbits/claude-code-config), [obra/superpowers](https://github.com/obra/superpowers), IIA/OWASP standards

### mental-models

Apply Charlie Munger's latticework of 35 mental models to any decision or problem. Structured multi-disciplinary analysis that surfaces blind spots, biases, and second-order effects.

- **35 models** across 7 disciplines (psychology, economics, biology, physics, engineering, mathematics, systems)
- **Full or brief mode** (`--brief` for key-questions-only)
- Source: Swedish Investor (Parts 1-6), Poor Charlie's Almanack

## Installation

```bash
# Clone the repo
git clone https://github.com/joonchungpersonal-dev/misc-skills.git

# Copy the skill(s) you want
cp -r misc-skills/skills/grill ~/.claude/skills/
cp -r misc-skills/skills/mental-models ~/.claude/skills/
```

Then invoke in Claude Code:
```
/grill
/mental-models "Should I accept this job offer?" --brief
```

## Requirements

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI
- Claude API access

## License

MIT
