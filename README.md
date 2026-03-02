# Miscellaneous Claude Code Skills

Standalone skills for [Claude Code](https://docs.anthropic.com/en/docs/claude-code) — code review and decision analysis.

## Skills

### grill

Extension of Boris Cherny's "grill me" [prompting technique](https://www.threads.com/@boris_cherny/post/DUMZxTWElFm) (from his Claude Code tips) into a formalized 2-agent skill. A Comprehensive Reviewer agent finds issues across 7 categories, then a GPTLens Critic agent validates each finding to eliminate false positives.

- **5C audit findings** (Condition, Criteria, Cause, Effect, Recommendation — IIA Standards 2410/2420)
- **OWASP risk scoring** (Likelihood x Impact, 0-9 scale)
- **GPTLens Auditor/Critic pattern** (Hu et al., IEEE TPS 2023)
- Built on: Boris Cherny's "grill me" technique, [trailofbits/claude-code-config](https://github.com/trailofbits/claude-code-config), [obra/superpowers](https://github.com/obra/superpowers), IIA/OWASP standards

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
