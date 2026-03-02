---
name: grill
description: Adversarial 2-phase code review with 5C findings, OWASP risk scoring, and GPTLens critic validation. Use before pushing code.
user-invocable: true
argument-hint: [files-or-scope]
---

# /grill — Adversarial Code Review (2-Phase)

Stress-test code changes using a Comprehensive Reviewer agent + GPTLens Critic agent. Produces 5C findings with OWASP risk scores, eliminates false positives via critic validation, logs results to the audit dashboard.

## Attribution & Sources

This skill was built on the work of others. Proper credit:

- **GPTLens Auditor/Critic pattern**: Hu et al., "Large Language Model-Powered Smart Contract Vulnerability Detection: New Perspectives," IEEE TPS (2023). arXiv:2310.01152. The two-phase diverge/converge approach (auditor generates findings, critic validates) to reduce false positives originates from this paper.
- **Trail of Bits security configuration**: [trailofbits/claude-code-config](https://github.com/trailofbits/claude-code-config). Security review patterns and hardened Claude Code configuration from the Trail of Bits security research team.
- **obra/superpowers**: [obra/superpowers](https://github.com/obra/superpowers). An agentic skills framework and software development methodology by Jesse Vincent that informed the skill file structure and agent orchestration patterns.
- **5C Audit Finding Format (IIA)**: The Condition-Criteria-Cause-Effect-Recommendation framework is a standard internal audit methodology associated with IIA Standards 2410/2420.
- **OWASP Risk Rating Methodology**: OWASP Foundation. The Likelihood x Impact scoring matrix is adapted from the OWASP Risk Rating Methodology (owasp.org/www-community/OWASP_Risk_Rating_Methodology).
- **Fagan Inspection limits**: The 400 LOC threshold references the SmartBear/Cisco study of 2,500 code reviews showing defect detection drops beyond that threshold, building on Fagan's original inspection methodology (1976).

**Modifications by Joon Chung**: Consolidated from a 5-agent parallel architecture (`/plus-audit`) into a streamlined 2-agent sequential architecture (Comprehensive Reviewer + Critic). Added panel-tagging system, dashboard logging integration, paper trail archiving, and the "Think & Verify" self-disproof step before reporting.

## Methodology

- **5C Audit Finding Format**: Condition, Criteria, Cause, Effect, Recommendation (from internal audit practice, commonly associated with IIA standards)
- **OWASP-Inspired Risk Rating**: Severity = Likelihood (0-3) x Impact (0-3), score 0-9 (simplified adaptation of the OWASP Risk Rating Methodology)
- **GPTLens Auditor/Critic Pattern**: Two-phase diverge/converge to eliminate false positives
- **Think & Verify Prompting**: Reviewer must attempt to disprove findings before reporting
- **Fagan Review Limits**: Files exceeding 400 LOC are flagged (defect detection drops beyond that threshold)

## Step 1: Identify files to audit

If `$ARGUMENTS` specifies files or directories, use those. Otherwise, run:
- `git diff --name-only HEAD` to find uncommitted changes
- `git diff --name-only HEAD~1` to include the latest commit

Deduplicate the file list. Read each changed file in full before launching agents.

**Fagan limit note**: If any single file exceeds 400 LOC, flag it in the report header.

## Step 2: Launch Comprehensive Reviewer agent

Launch ONE agent using the Task tool with `subagent_type: general-purpose`. This agent covers ALL 7 review categories, pre-tagging each finding by panel.

### Agent 1: Comprehensive Reviewer

The prompt for this agent must include the full contents of every file being audited, then the following instructions:

```
You are a skeptical senior code reviewer. Your job is to find real problems, not praise. Review every file above across ALL 7 categories below. Pre-tag each finding with its PANEL for downstream sorting.

CATEGORIES (with panel tags):

[PANEL: security] 1. SECURITY & PRIVACY
- Hardcoded secrets, API keys, tokens, PII exposure
- Injection vulnerabilities (SQL, command, template, XSS)
- Missing input validation or sanitization
- Rate limiting bypass, TOCTOU race conditions
- Insecure randomness, weak hashing
- Missing authentication/authorization checks

[PANEL: logic] 2. CORRECTNESS
- Off-by-one errors, boundary conditions
- None/null/empty handling, missing data paths
- Incorrect boolean logic, operator precedence
- Type mismatches, silent type coercion bugs

[PANEL: logic] 3. EDGE CASES
- Empty/large/Unicode inputs, concurrent access
- Network failures, timeout handling
- Exception paths that leave state inconsistent
- Test assertions that always pass (tautologies)

[PANEL: scalability] 4. PERFORMANCE
- N+1 queries, unbounded loops, missing pagination
- Memory leaks, unbounded data structures
- Blocking calls in async context
- O(n^2) or worse on non-trivial data sizes
- Missing timeouts on network/LLM calls

[PANEL: logic] 5. BREAKING CHANGES
- API contract violations, backward compatibility
- Database migration issues, schema changes
- Removed or renamed public interfaces

[PANEL: ux] 6. TEST COVERAGE
- Are changes tested? What cases are missing?
- Dead code, unreachable branches
- Note: UX panel caps at HIGH severity (no CRITICAL)

[PANEL: ux] 7. CODE QUALITY
- Unclear error messages, broken formatting
- Unused imports, dead variables, commented-out code
- Inconsistent naming or patterns
- Note: UX panel caps at HIGH severity (no CRITICAL)

METHODOLOGY — For EACH finding:

A) Think & Verify: Before reporting, attempt to DISPROVE it. Check if the code has guards, exception handlers, upstream validation, or framework guarantees. Only report findings that survive your counter-evidence check.

B) Use 5C format:
- **Condition**: What IS happening (quote the specific code)
- **Criteria**: What SHOULD be happening (cite OWASP/CWE where applicable)
- **Cause**: WHY the gap exists
- **Effect**: What's the IMPACT if exploited/triggered
- **Recommendation**: Exact fix with code suggestion

C) Score using OWASP risk rating:
- Likelihood (0-3): How likely to occur? (0=theoretical, 1=difficult, 2=moderate, 3=trivial/every-invocation)
- Impact (0-3): How bad? (0=negligible, 1=minor, 2=significant/data-loss, 3=catastrophic/RCE)
- Risk Score = Likelihood x Impact (0-9)
- Map: 7-9=CRITICAL, 4-6=HIGH, 2-3=MEDIUM, 0-1=LOW
- Exception: UX panel findings cap at HIGH

D) Assign confidence score (0-100): how certain after counter-evidence check?

FORMAT each finding as:

**[PANEL: xxx] [SEVERITY] (Risk: LxI=Score, Confidence: N%)** file:line
- Condition: ...
- Criteria: ...
- Cause: ...
- Effect: ...
- Recommendation: ...
```

## Step 3: Launch Critic agent (GPTLens pattern)

After Agent 1 returns, launch a 2nd agent using the Task tool with `subagent_type: general-purpose`. This agent validates every finding to eliminate false positives.

### Agent 2: GPTLens Critic

The prompt must include the full file contents AND the raw findings from Agent 1:

```
You are a senior code reviewer acting as a Critic. Your job is to VALIDATE or REJECT each finding from the prior reviewer. LLM reviewers produce false positives — your job is to catch them.

Here are the files being reviewed:
[FULL FILE CONTENTS]

Here are the raw findings:
[AGENT_1_FINDINGS]

For EACH finding:
1. Re-read the actual code at the cited file:line
2. Is this a real issue or a hallucinated concern?
3. Does the code already handle this via guards, try/except, upstream validation, or framework guarantees?
4. Is the severity rating appropriate? Re-score using Likelihood x Impact if needed
5. Is the confidence score reasonable?
6. Is this a duplicate of another finding? If so, merge them

Produce a validated findings list. For each finding, assign a verdict:
- **CONFIRMED**: Real issue. Keep it (optionally adjust severity/confidence)
- **DOWNGRADED**: Real but over-scored. Adjust severity downward with explanation
- **REJECTED**: False positive. Explain why with code evidence
- **MERGED**: Duplicate of another finding. Specify which one it merges into

Output the validated list preserving the 5C format and panel tags for all confirmed/downgraded findings.

At the end, report summary stats:
- Total findings in
- Confirmed out
- Rejected (with reasons)
- Downgraded (with new scores)
- Merged (with merge targets)
```

## Step 4: Consolidate findings

After the Critic returns, combine validated findings into a single prioritized report:

1. **Remove rejected findings** — drop anything the Critic flagged as false positive
2. **Merge duplicates** — combine findings the Critic identified as overlapping
3. **Sort by risk score** — highest first, then by confidence
4. **Split into two sections:**
   - **Actionable Now** — CRITICAL and HIGH findings, plus MEDIUM with confidence >80%
   - **Deferred** — LOW findings, MEDIUM with confidence <80%, and downgraded items

**Report header** must include:
- Files audited (count and names)
- Fagan limit warnings (files >400 LOC)
- False positive rate (rejected / total raw findings)
- Validation summary (confirmed / downgraded / rejected / merged)

Present the consolidated report and ask: **"Want me to fix the actionable items?"**

## Step 5: Run tests

After presenting findings, auto-detect and run the project's test suite:
- If `pytest` is available (Python project): `python -m pytest tests/ -v` (activate venv first if present)
- If `package.json` exists with test script: `npm test`
- If `Makefile` has test target: `make test`
- If `Cargo.toml` exists: `cargo test`
- If no test runner found, note "No test runner detected" in the report

Report test results alongside audit findings.

## Step 6: Log audit results

After the audit completes (findings presented, tests run), log the results.

### 6a: Gather metadata

Run these commands to collect project info:
- `git remote get-url origin` → extract repo name (fallback: directory basename)
- `pwd` → project_path
- `git branch --show-current` → branch
- `git rev-parse --short HEAD` → commit_sha

### 6b: Construct the log entry

Build a JSON object with this schema:

```json
{
  "id": "<ISO-timestamp>",
  "timestamp": "<ISO-8601 UTC>",
  "project": "<repo-name>",
  "project_path": "<absolute-path>",
  "branch": "<current-branch>",
  "commit_sha": "<HEAD-short-SHA>",
  "scope": "<description of what was audited>",
  "files_audited": "<number>",
  "fagan_warnings": "<count of files >400 LOC>",
  "panels": {
    "security":    {"critical": 0, "high": 0, "medium": 0, "low": 0},
    "logic":       {"critical": 0, "high": 0, "medium": 0, "low": 0},
    "ux":          {"high": 0, "medium": 0, "low": 0},
    "scalability": {"critical": 0, "high": 0, "medium": 0, "low": 0}
  },
  "totals": {"critical": 0, "high": 0, "medium": 0, "low": 0, "total": 0},
  "validation": {
    "raw_findings": "<total before critic>",
    "confirmed": "<count>",
    "downgraded": "<count>",
    "rejected": "<count>",
    "merged": "<count>",
    "false_positive_rate": "<rejected / raw_findings as percentage>"
  },
  "disposition": {"actioned": "<count>", "deferred": "<count>"},
  "tests": {"passed": "<count>", "failed": "<count>"},
  "findings": [
    {
      "panel": "security|logic|ux|scalability",
      "severity": "CRITICAL|HIGH|MEDIUM|LOW",
      "risk_score": "<0-9>",
      "confidence": "<0-100>",
      "file": "path/to/file",
      "line": "<line number>",
      "condition": "What IS happening",
      "criteria": "What SHOULD be happening",
      "cause": "WHY the gap exists",
      "effect": "IMPACT if not fixed",
      "recommendation": "Exact fix",
      "critic_verdict": "confirmed|downgraded|merged",
      "status": "actioned|deferred"
    }
  ]
}
```

### 6c: Append to audit-log.json

Read `~/.claude/audit-log.json`. If it exists, parse the JSON array and append the new entry. If it doesn't exist, create it with `[entry]`. Write the updated array back.

**Important**: This file is append-only. Never overwrite or remove existing entries.

### 6d: Save full paper trail

Create a timestamped directory:

```
~/.claude/audit-trails/grill/<YYYY-MM-DD>_<HH-MM-SS>_<project>/
  metadata.json          # The JSON entry from 6b
  agent1_reviewer.md     # Full raw output from the Comprehensive Reviewer
  agent2_critic.md       # Full raw output from the GPTLens Critic
  consolidated_report.md # The final report as presented to the user
  files_audited.txt      # List of files with their line counts
  diff_applied.patch     # (if fixes were applied) git diff of changes made
```

Steps:
1. Create the directory: `mkdir -p ~/.claude/audit-trails/grill/<timestamp>_<project>`
2. Write `metadata.json` — the full JSON log entry
3. Write each agent's raw output to its `.md` file
4. Write the consolidated report to `consolidated_report.md`
5. Write the list of audited files with line counts to `files_audited.txt`
6. If fixes were applied, capture `git diff` and save as `diff_applied.patch`
7. Commit the new trail to the audit-trails git repo:
   ```
   cd ~/.claude/audit-trails && git add -A && git commit -m "grill: <project> <timestamp>"
   ```

## Step 7: Regenerate dashboard data

After updating `audit-log.json`, update the inline data block in the dashboard HTML:

1. Read the full contents of `~/.claude/audit-log.json`
2. Read `~/.claude/audit-dashboard/index.html`
3. Replace everything between `<script id="audit-data">` and the next `</script>` with:
   ```
   <script id="audit-data">
   // AUTO-GENERATED by /grill Step 7 — do not edit manually
   const AUDIT_DATA = <full JSON array from audit-log.json>;
   </script>
   ```
4. Write the updated `index.html` back
5. Tell the user: **"Dashboard updated. Open `~/.claude/audit-dashboard/index.html` in your browser (Code Review tab)."**
