# Miscellaneous Claude Code Skills

Standalone skills for [Claude Code](https://docs.anthropic.com/en/docs/claude-code).

## Skills

### mental-models

Apply Charlie Munger's latticework of 35 mental models to any decision or problem. Structured multi-disciplinary analysis that surfaces blind spots, biases, and second-order effects.

- **35 models** across 7 disciplines (psychology, economics, biology, physics, engineering, mathematics, systems)
- **Full or brief mode** (`--brief` for key-questions-only)
- Source: Swedish Investor (Parts 1-6), Poor Charlie's Almanack

## Installation

```bash
# Clone the repo
git clone https://github.com/joonchungpersonal-dev/misc-skills.git

# Copy the skill
cp -r misc-skills/skills/mental-models ~/.claude/skills/
```

Then invoke in Claude Code:
```
/mental-models "Should I accept this job offer?" --brief
```

## Requirements

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI
- Claude API access

## License

MIT
