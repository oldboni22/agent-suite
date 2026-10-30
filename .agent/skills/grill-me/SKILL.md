---
name: grill-me
description: Grill the user relentlessly about a plan, decision, or idea. Use when the user wants to stress-test their thinking, or uses any "grill" trigger phrases.
---

# Grill me

Interview the user relentlessly until you reach a shared understanding. Map the discussion as a design tree where every decision branches into dependent decisions.

Work the tree sequentially. The frontier contains decisions whose prerequisites are already settled. Ask questions one at a time. This avoids cognitive overload and allows each answer to reshape the remaining tree.

## Question format

Format each question with background, concrete options, a direct question statement, and a recommended answer:

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

1. **One decision at a time**: Present one question per turn. Wait for the user response before advancing the tree.
2. **Word for consent**: Word each question so that answering "yes" accepts the recommended answer. The user can also pick an alternative option or provide custom guidance.
3. **Explore before asking**: Finding facts is your job, never the user's. Search codebases, inspect configurations, or check documentation directly. Never ask the user for information you can discover yourself.
4. **Dynamic frontier**: Each user answer settles one branch and unblocks dependent decisions. Recompute the frontier after each answer.
5. **Termination**: The interview ends when the frontier is empty. Every architectural choice, boundary, and trade-off must be settled. Do not proceed to implementation or file creation until the user confirms you have reached a shared understanding.
