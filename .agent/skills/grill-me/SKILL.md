---
name: grill-me
description: Grill the user relentlessly about a plan, decision, or idea. Use when the user wants to stress-test their thinking, or uses any "grill" trigger phrases.
---

# Grill me

Interview the user relentlessly until you reach a shared understanding. Map this as a design tree: every decision branches into the decisions that hang off it.

Work the tree in rounds. The frontier is every decision whose prerequisites are already settled: the questions you can ask now without guessing at answers you have not heard yet. Ask the whole frontier in one round: number each question and give your recommended answer. Then wait for the user answers before the next round.

Format a round like so:

```markdown
Q1: <question title>
<question body, including background and options>

Recommended answer: <your recommended answer>

---

Q2: <question title>
<question body, including background and options>

Recommended answer: <your recommended answer>
```

Word each question so answering "yes" accepts your recommended answer.

Each round the user answers reshapes the tree. Settled decisions push the frontier outward and unblock questions that depended on them. Recompute the frontier and ask the next round. A question whose answer depends on another question still open in this round belongs to a later round, not this one.

Finding facts is your job, never the user's. When a frontier question needs a fact from the environment or codebase, explore it directly or dispatch a sub-agent to find it. Do not ask the user for anything you can look up yourself. Unsettled prerequisites wait for exploration to finish, while independent questions in the frontier can be asked immediately. The decisions belong to the user: put each decision to them and wait.

The session finishes when the frontier is empty: every branch of the design tree is visited, and nothing is left silently assumed. Do not act on the plan until the user confirms you have reached a shared understanding.
