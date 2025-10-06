# Architect/Implementer Workflow for AI Coding Agents

This document describes Jesse Vincent's proven methodology for using AI coding agents effectively, adapted for our workflow.

## Overview

**Core Concept:** Separate the design phase (architect) from the implementation phase (implementer) using multiple AI sessions and written plans.

**Why This Works:**
- Reduces review burden by 50%+ (you review execution vs design decisions)
- Eliminates "why did you do it that way?" questions
- Enables running 2-3+ parallel features without confusion
- Creates reusable plan templates

---

## The Four Key Prompts

### 1. Brainstorming Prompt (Architect Session)

**When to use:** Starting a new feature or complex task

**The Prompt:**
```
I've got an idea I want to talk through with you. I'd like you to help me turn it into a fully formed design & spec (& eventually an implementation plan).

Check out the current state of the project in our working directory to understand where we're starting off, then ask me questions, one at a time, to help refine the idea.

Ideally, the questions would be multiple choice, but open-ended questions are OK, too. Don't forget: only one question per message.

Once you believe you understand what we're doing, stop & describe the design to me, in sections of maybe 200-300 words at a time, asking after each section whether it looks right so far.
```

**Key Points:**
- Limits responses to 200-300 words (prevents walls of text you won't read)
- One question at a time (forces engagement)
- No code writing during this phase

---

### 2. Planning Prompt (Architect Session)

**When to use:** After design is approved, before implementation

**The Prompt:**
```
Great. I need your help to write out a comprehensive implementation plan.

Assume that the engineer has zero context for our codebase & questionable taste. Document everything they need to know: which files to touch for each task, code, testing, docs they might need to check, how to test it. Give them the whole plan as bite-sized tasks. DRY. YAGNI. TDD. Frequent commits.

Assume they are a skilled developer, but know almost nothing about our toolset or problem domain. Assume they don't know good test design very well.

Please write out this plan, in full detail, into ~/Documents/coding/plans/YYYY-MM-DD/[feature-name].md
```

**Plan File Must Include:**
- **Bite-sized tasks** (1-4 hours each)
- **Exact file paths** to modify
- **Code examples/snippets** for each task
- **Testing approach** per task
- **Success criteria** for each task
- **Suggested commit messages**

---

### 3. Implementer Prompt (New Session)

**When to use:** Starting implementation in a fresh Claude session

**The Prompt:**
```
Please read ~/Documents/coding/plans/YYYY-MM-DD/[feature-name].md & [design-doc if separate]. Let me know if you have questions.
```

**Then, after review:**
```
Please execute the first 3-4 tasks. If you have questions, please stop & ask me. DO NOT DEVIATE FROM THE PLAN.
```

**Key Behaviors:**
- Implementer has zero context pollution from brainstorming
- Must ask before deviating from plan
- Updates plan file with progress after each chunk
- Uses `*/compact*` and `*/clear*` to reduce context between chunks

---

### 4. Architect Review Prompt (Original Session)

**When to use:** After implementer completes a chunk of tasks

**The Prompt:**
```
The implementer says it's done tasks 1-3. Please check the work carefully.
```

**Architect Reviews For:**
- Did they follow the spec exactly?
- Are tests adequate?
- Any bugs or edge cases missed?
- Does it align with original design intent?

**Your Role:** Copy feedback between sessions as "PM"

---

## Standard Operating Procedure

### Phase 1: Design (Architect Session)

1. **Start:** Use brainstorming prompt
2. **Refine:** Answer questions one at a time
3. **Review:** Approve design in 200-300 word chunks
4. **Plan:** Use planning prompt to create `~/Documents/coding/plans/YYYY-MM-DD/feature.md`

### Phase 2: Implementation (New Implementer Session)

1. **New tab/window:** Fresh Claude Code session
2. **Load plan:** Use implementer prompt with plan file path
3. **Execute:** "Do tasks 1-3, don't deviate from plan"
4. **Chunk work:** Do 3-4 tasks at a time, then review

### Phase 3: Review Loop

1. **Switch to architect tab**
2. **Architect reviews** implementer's work
3. **You play PM:** Copy feedback between sessions
4. **Implementer fixes** issues
5. **Architect approves** → implementer updates plan → next chunk

### Phase 4: Finalize

1. Implementer pushes to GitHub
2. Creates pull request
3. (Optional) CodeRabbit review with role-play prompt

---

## Git Worktrees for Parallel Work

**Setup (first time):**
```bash
cd ~/Documents/coding/your-project
mkdir .worktrees
```

**For each parallel feature:**
```bash
cd ~/Documents/coding/your-project/.worktrees
git worktree add feature-name
cd feature-name
npm install  # or project setup
npm test     # verify clean baseline
claude       # start implementer session
```

**Result:** Run 3-4 features in parallel, each with:
- Own worktree directory
- Own plan in `~/Documents/coding/plans/YYYY-MM-DD/`
- Own implementer session
- Shared architect session for all reviews

---

## Plan Directory Structure

```
~/Documents/coding/plans/
├── 2025-10-06/
│   ├── podcast-search.md
│   ├── blog-generator.md
│   └── email-classifier.md
├── 2025-10-07/
│   └── api-integration.md
└── templates/
    ├── feature-template.md
    └── api-integration-template.md
```

**Auto-create dated directories:** Yes, Claude can create these automatically when writing plans

---

## CodeRabbit Integration (Optional)

**After PR is created, use this prefix for CodeRabbit reviews:**

```
A reviewer did some analysis of this PR. They're external, so reading the codebase cold. This is their analysis of the changes & I'd like you to evaluate the analysis & the reviewer carefully.

1) Should we hire this reviewer?
2) Which of the issues they've flagged should be fixed?
3) Are the fixes they propose the correct ones?

Anything we *should* fix, put on your todo list.
Anything we should skip, tell me about now.

[PASTE CODERABBIT REVIEW HERE]
```

**This makes Claude critically evaluate suggestions instead of blindly implementing them.**

---

## Your Role as "Product Manager"

**Between Sessions You:**
- ✅ Copy architect feedback → implementer
- ✅ Copy implementer questions → architect
- ✅ Make final call on disputes
- ✅ Approve work before moving to next chunk
- ✅ Ensure plan stays updated

**You Don't:**
- ❌ Write code yourself
- ❌ Debug implementation details
- ❌ Explain architecture (it's in the plan)

---

## Success Metrics

**What We're Optimizing For:**
- 50%+ reduction in review time
- Fewer design debates during implementation
- Ability to run 2-3 parallel features
- Reusable plan templates

**Red Flags:**
- Architect & implementer disagreeing on approach → plan wasn't clear enough
- Implementer deviating from plan → plan was wrong or incomplete
- Taking longer to plan than implement → tasks too small

---

## Quick Start Checklist

**First Time Setup:**
- [ ] Create `~/Documents/coding/plans/` directory
- [ ] Bookmark this CLAUDE.md file
- [ ] Test with one small feature

**For Each New Feature:**
- [ ] Open architect session → brainstorm
- [ ] Architect writes plan to `~/Documents/coding/plans/YYYY-MM-DD/feature.md`
- [ ] Review & approve plan
- [ ] Open fresh implementer session → load plan
- [ ] Implement in 3-4 task chunks
- [ ] Architect reviews each chunk
- [ ] Push & create PR when done

---

## Common Patterns

### Pattern 1: Research Tasks
**Use Case:** "Can library X integrate with our stack?"

```
Architect: Research & write findings to plans/YYYY-MM-DD/research-X.md
(No implementer needed - just research output)
```

### Pattern 2: Multiple Small Features
**Use Case:** 3 independent bug fixes

```
One architect session creates 3 plans
Three implementer sessions (parallel worktrees)
Architect reviews all three in rotation
```

### Pattern 3: Large Refactor
**Use Case:** Rewrite authentication system

```
Architect: Create master plan with phases
Implementer 1: Phase 1 (data layer)
Architect reviews & approves
Implementer 2: Phase 2 (API layer)
Architect reviews & approves
Implementer 3: Phase 3 (UI layer)
```

---

## Troubleshooting

**Problem:** Implementer wants to deviate from plan
**Solution:** Tell it to ask architect first, copy question to architect session

**Problem:** Architect & implementer give conflicting advice
**Solution:** You decide. Update plan to reflect decision.

**Problem:** Plan is too vague
**Solution:** Architect session: "Expand task 3 with specific file paths & code examples"

**Problem:** Running out of context in implementer
**Solution:** Use `*/compact*` and `*/clear*` between task chunks

**Problem:** Forgot which session is which
**Solution:** Name terminal tabs "Architect" & "Implementer-FeatureName"

---

## Advanced: Template System

**Create reusable templates in `~/Documents/coding/plans/templates/`:**

**Example: API Integration Template**
```markdown
# API Integration: [SERVICE_NAME]

## Tasks
1. Research [SERVICE] API docs
   - Files: docs/research/[service].md
   - Success: Document auth flow, rate limits, endpoints

2. Create API client
   - Files: src/integrations/[service]/client.ts
   - Tests: src/integrations/[service]/client.test.ts
   - Success: Can authenticate & make basic request

3. Implement core endpoints
   - Files: src/integrations/[service]/endpoints.ts
   - Tests: Mock responses, error handling
   - Success: 90%+ test coverage

4. Add error handling & retries
   - Files: Update client.ts with retry logic
   - Tests: Test timeout, network errors, rate limits
   - Success: Graceful degradation on all error types
```

**Use template:**
```
Architect: Use ~/Documents/coding/plans/templates/api-integration.md as a base.
Customize for [Stripe API integration] & write to plans/2025-10-06/stripe-integration.md
```

---

## Credits

This methodology is based on Jesse Vincent's ["How I'm using coding agents in September, 2025"](https://blog.fsck.com/2025/10/05/how-im-using-coding-agents-in-september-2025/), with adaptations for our workflow.

Referenced in Simon Willison's ["Embracing the parallel coding agent lifestyle"](https://simonw.substack.com/p/embracing-the-parallel-coding-agent).
