---
name: dev-reflection
description: |
  After fixing a bug, resolving an incident, or completing a non-trivial debugging session, generate a reflection markdown file that captures what happened, what was tried, what worked, and key takeaways.
  Use this skill whenever: a bug fix took multiple attempts, the initial approach was wrong or over-engineered, a debugging investigation had multiple phases, or a surprising root cause was discovered.
  Also use when the user says "write a reflection", "document what happened", "capture this fix", or "save learnings from this".
  The output should match the structure of existing reflection files (see references/) — concise, evidence-driven, with a tone of honest self-assessment rather than self-praise.
---

# Dev Reflection Generator

After resolving a development problem, generate a reflection document that captures the investigation, the mistakes made, the fix applied, and reusable learnings.

## When to write a reflection

Not every fix needs one. Write a reflection when:

- The bug took 2+ attempts or investigation phases to solve
- The root cause was surprising or non-obvious
- The initial approach was wrong, incomplete, or over-engineered
- A framework/dependency behaved differently than documented
- The solution has broader lessons for the team or future work

Skip for: typos, trivial config changes, obvious one-line fixes.

## Choose the right structure

There are three reflection patterns. Pick the one that matches what actually happened.

### Pattern A: Investigation (multi-phase debugging)

Use when the problem required runtime investigation with evidence gathering across phases.

```markdown
# [Brief title describing the problem] (reflection)

## What happened?

[2-3 sentences: what symptom did the user see, in what context?]

## How we found the problem

We treated this as a runtime investigation, not a code-only guess.

### Phase 1: [Phase name - what you suspected first]

[What instrumentation or logging was added. What specific things were monitored.]

**Evidence:** [What the logs/data actually showed. Be specific — include actual values, event names, absence of events.]

### Phase 2: [Phase name - what you discovered next]

[What the second round of investigation revealed.]

**Evidence:** [Concrete findings that pointed to the real cause.]

## What we changed

### [Category of changes, e.g., "Making X work (platform)"]

- **[Component or layer]:** [What was changed and why. One sentence.]
- **[Component or layer]:** [Another change.]

### [Category of changes, e.g., "Making Y reliable (store)"]

- **[Component or layer]:** [What was changed.]

## Why did this happen?

1. **[Root cause 1]:** [Why this caused the symptom. Be specific about the mechanism.]
2. **[Root cause 2]:** [If multiple factors compounded, explain each.]
3. **[Systemic cause]:** [Why this wasn't caught earlier — missing tests, framework defaults, split responsibilities.]

## Takeaways

- **[Actionable lesson 1]** — [Why this matters, how to apply it next time.]
- **[Actionable lesson 2]** — [Generalize beyond this specific bug.]
```

See [`references/sidebar-drag-drop-bug-fix.md`](references/sidebar-drag-drop-bug-fix.md) for a real example.

### Pattern B: Over-engineering correction

Use when the initial approach was overly complex and a simpler commit/approach won.

```markdown
# [Brief title] (reflection)

## What happened?

I was tasked with [actual task, e.g., "fixing failing Playwright E2E tests"]. The issues were:
1. [Specific failure 1]
2. [Specific failure 2]
3. [Specific failure 3]

## What I did (over-engineered approach)

I made extensive modifications to `[file]`:

```[language]
// My version - [describe the complexity]
[show the over-engineered code/config]
```

I also added:
- [unnecessary thing 1]
- [unnecessary thing 2]
- [unnecessary thing 3]

## What the commit did (simplified approach)

The commit took a minimal, pragmatic approach:

```[language]
// Commit version - [describe the simplicity]
[show the working minimal version]
```

The commit:
- [What it kept - essential only]
- [What it removed]
- [What it actually added vs. what was planned]

## Why the simpler approach is better

1. **YAGNI**: [Why the extra complexity wasn't needed. Quantify if possible — e.g., "72 tests × 6 browsers = 432 executions".]
2. **Reduced maintenance surface**: [Every added config is a future bug.]
3. **No premature abstraction**: [What conditionals or abstractions were speculative.]
4. **Fewer points of failure**: [What platform issues were avoided by not including them.]
5. **Faster feedback loop**: [How minimalism speeds up iteration.]

## My fixes (that were valuable)

The changes I made that were actually necessary:
- [Genuinely useful fix 1]
- [Genuinely useful fix 2]
- [Genuinely useful fix 3]

## What I should have done

1. Ask: "[Question I should have asked]" before [action I took without asking]
2. Question: "[Assumption I should have challenged]" before [work based on that assumption]
3. Consider: "[Minimum viable approach]" and start there
4. Recognize: "[What the real task was]" vs "[What I thought the task was]"
```

See [`references/playwright-e2e-setup.md`](references/playwright-e2e-setup.md) for a real example.

### Pattern C: Straightforward bug fix

Use when the root cause was identified cleanly and fixed in one pass.

```markdown
# [Brief title]

## Problem

When [action the user took], the operation failed with:
```
[exact error message]
```

## Root Cause

The bug was caused by [concise root cause statement].

### [Relevant mechanism — e.g., how the framework works]

[Explain the framework/layer behavior that caused confusion. Use a table if comparing versions or formats.]

| Side A | Side B |
|--------|--------|
| [relevant detail] | [relevant detail] |

### What Went Wrong

[The specific mistake or misunderstanding. If there was a wrong "fix" that made things worse, show the wrong code vs. right code.]

```[language]
// WRONG - [what was done and why it was wrong]
[bad code]

// CORRECT - [what should have been done]
[good code]
```

## Solution

[Describe the fix. Include code if relevant.]

## Files Changed

1. `[path]` - [what was changed]
2. `[path]` - [what was changed]

## Lessons Learned

1. **[Lesson 1]** - [How the error message or logs revealed the real issue.]
2. **[Lesson 2]** - [Framework-specific insight.]
3. **[Lesson 3]** - [General principle, not just this bug.]
```

See [`references/release-pipeline.md`](references/release-pipeline.md) or [`references/environment-creation-bug-fix.md`](references/environment-creation-bug-fix.md) for real examples.

## Writing rules

### Tone

- **Honest self-assessment**, not self-praise. Say "I over-engineered" not "I explored multiple approaches."
- **Evidence over assertion.** "Logs showed no `drop` events" not "The drop handler wasn't working."
- **Specific values and names.** Quote exact error messages, log lines, config keys.
- **No filler.** Every sentence should teach something or document something useful.

### What to include

- **Exact error messages** — they're often more informative than you think
- **Evidence from investigation** — what the logs/data actually showed, not just what you suspected
- **Wrong approaches** — document the dead ends, not just the fix. The wrong path teaches more than the right one.
- **Quantified impact** — "72 tests × 6 browsers = 432 executions" hits harder than "lots of tests"
- **Framework version context** — if a framework upgrade caused the issue, name the versions

### What to skip

- Generic statements like "debugging is important" or "test thoroughly"
- Praising the final commit author
- Restating obvious coding practices everyone knows
- The full git diff — link to it or show only the relevant lines
- Blame or finger-pointing

### File naming

Save reflections in the project's docs folder with kebab-case names ending in `-bug-fix.md` or `*-reflection.md`:

```
docs/
├── sidebar-drag-drop-bug-fix.md
├── playwright-e2e-setup.md       # reflection-style naming
├── release-pipeline-debugging.md
```
