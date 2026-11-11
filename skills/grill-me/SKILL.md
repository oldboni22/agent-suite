---
name: grill-me
description: Grill the user relentlessly about a plan, decision, or idea. Use when the user wants to stress-test their thinking, or uses any "grill" trigger phrases.
---

# Grill me

Interview the user relentlessly until you reach a shared understanding. Map the discussion as a design tree where every decision branches into dependent decisions.

Work the tree sequentially. The frontier contains decisions whose prerequisites are already settled. Ask questions one at a time. This avoids cognitive overload and allows each answer to reshape the remaining tree.

## Lifecycle and Activation
- **Topic-persistent**: Once activated, `grill-me` mode governs all interview turns until the frontier is cleared and a plan is produced.
- **Do not re-invoke**: Do not re-call `activateSkill` on each question turn.

## Modes of engagement

### 1. Discussion and architectural sparring mode
When the user's response is exploratory, indecisive, asks "what do you think?", or pushes back on a premise:
- Shift into an architectural sparring partner.
- Discuss trade-offs, edge cases, and real-world implications directly and concisely.
- Do not modify or branch the design tree prematurely while in active discussion.
- Do not force multiple-choice options or binary consent during an ongoing exchange.
- Only when a natural stopping point or consensus in the discussion is reached, formulate the settled choice or next unresolved decision as a structured question.

### 2. Structured frontier mode
When presenting a new unresolved decision on the frontier or locking in a discussed direction:
- Present exactly one question per turn using the structured format below.
- Format each question with background, concrete options, a direct question statement, and a recommended answer.

## Question format

```markdown
### Q<number>: <question title>

**Background**:
<1 to 2 sentences explaining the context, problem, or trade-off.>

**Options**:
- **Option A (Recommended)**: <concrete description of recommended choice>
- **Option B**: <alternative choice>
- **Option C**: <alternative choice or "None">

**Question**:
<Direct question worded so answering "yes" accepts the recommended option.>

**Recommended answer**:
Yes. <1 to 2 sentences explaining why Option A is chosen.>
```

## Interview rules

1. **One decision at a time**: Present one structured question per turn when on the frontier. Wait for the user response before advancing the tree.
2. **Word for consent in frontier mode**: Word each structured question so that answering "yes" accepts the recommended answer. The user can also pick an alternative option, provide custom guidance, or open a discussion.
3. **No premature tree mutations**: While discussing a branch, do not alter the broader decision tree until the trade-off is settled.
4. **Explore before asking**: Finding facts is your job, never the user's. Search codebases, inspect configurations, or check documentation directly. Never ask the user for information you can discover yourself.
5. **Dynamic frontier**: Each settled user answer resolves one branch and unblocks dependent decisions. Recompute the frontier after each answer.
6. **Bounded scope**: Frontier questions must be bounded to the current milestone (maximum 3 to 5 key decisions). Defer downstream implementation questions that belong to coding or testing.
7. **Termination**: The interview ends when the frontier is empty. Every architectural choice, boundary, and trade-off must be settled. Do not proceed to implementation or file creation until the user confirms you have reached a shared understanding.
