# Technique Decision Tree

Read this file when you are about to select a technique and want to avoid choosing by inertia. Walk the tree top-down, stop at the first YES, apply the technique at that node. If no node matches, deliver a direct prompt with no scaffolding.

All techniques listed here are **safe for single-prompt execution**. Techniques that fabricate in single-prompt form (Mixture of Experts, Tree of Thought, Graph of Thought, Universal Self-Consistency, Meta-Prompting with simulated experts, Prompt Chaining as embedded instruction) are excluded — see the Hard Rules in SKILL.md.

---

## The tree

```
Q1. Is the target model a reasoning-native model?
    (o3, o4-mini, DeepSeek-R1, Qwen3 thinking mode)
    ├── YES → Zero-shot, short, clean. No CoT, no role padding, no scaffolding.
    │         Stop here. State goal + output format only.
    └── NO ↓

Q2. Must the output be machine-parseable?
    (JSON, XML, CSV, schema-constrained)
    ├── YES → Structured Output lock.
    │         Specify exact schema. Forbid prose outside the structure.
    │         Define fallback for unfillable fields.
    └── NO ↓

Q3. Is this a logic, math, debugging, or multi-variable analysis task
    on a standard (non-reasoning-native) model?
    ├── YES → Chain of Thought.
    │         "Think through this step by step before answering."
    │         Only on Claude, GPT-5.x, Gemini, Qwen2.5, Llama, Mistral.
    └── NO ↓

Q4. Is the desired format easier to show than describe
    AND has the user already failed to get it without examples?
    ├── YES → Few-shot (2–5 examples).
    │         Examples must be representative of the real case, not generic.
    └── NO ↓

Q5. Is this an optimization of an existing prompt that failed?
    ├── YES → Auto-Refinement.
    │         Require: original prompt + actual failed output + what specifically failed.
    │         Output: root-cause classification + rewritten prompt + changelog.
    └── NO ↓

Q6. Is this exploration where contrasting perspectives add value?
    (strategy options, design alternatives, trade-off analysis)
    ├── YES → Multi-Vector.
    │         Generate 2–3 labeled contrasting approaches.
    │         Each: objective, method, pros, cons, ideal use condition.
    │         Not an expert panel — a single model listing alternatives.
    └── NO ↓

Q7. Is this a factual, research, or citation task?
    ├── YES → Grounding anchor.
    │         "Use only information you are highly confident is accurate.
    │          If uncertain, write [uncertain] next to the claim.
    │          Do not fabricate citations or statistics."
    └── NO ↓

Q8. Is this an agentic task (Claude Code, Cursor, Cline, Devin, Antigravity)?
    ├── YES → Agentic scaffold.
    │         Starting state + target state + allowed actions + forbidden actions
    │         + stop conditions + checkpoint output + human review triggers.
    └── NO ↓

DEFAULT → Direct prompt. No scaffolding.
          Role + task + output format + constraints. Nothing more.
```

---

## Node reference — exact phrasing to embed

### Q1 — Reasoning-native model

Do not add any of these: "think step by step", "reason carefully", "show your work", "break this down", role padding like "you are a brilliant analyst who considers every angle". These actively degrade output on o3, o4-mini, R1, Qwen3-thinking.

What to write instead:

```
Task: [precise operation]
Output: [exact format and length]
[Input data or context, if any]
```

That is the entire prompt. Reasoning-native models fail gracefully on sparse prompts and degrade on dense ones.

---

### Q2 — Structured Output

```
Your response MUST be valid [JSON / XML / CSV] matching this exact structure:
[schema or example]

Do not include markdown fences, preamble, or explanation outside the structure.
If a field cannot be filled from available data, use null and add a sibling key
"<field>_missing_reason" with a one-sentence explanation.
```

---

### Q3 — Chain of Thought (standard models only)

```
Think through this step by step before answering.
After reasoning, provide the final answer under a "## Answer" heading.
```

Do not use on reasoning-native models (see Q1).

---

### Q4 — Few-shot

```
Examples:

Input: [representative case 1 from the user's actual domain]
Output: [expected result 1, exactly as it should appear]

Input: [representative case 2]
Output: [expected result 2]

Now process:
Input: [real case]

Follow the format of the examples precisely.
```

Examples must come from the user's real domain. Generic examples teach generic output.

---

### Q5 — Auto-Refinement

Require three inputs from the user before applying this node:

1. Original prompt (verbatim)
2. Output that failed (verbatim, not a paraphrase)
3. What specifically failed — wrong format, hallucination, missed constraint, misread intent

Then classify the root cause into exactly one of:

- **Ambiguity** — prompt admits more than one valid interpretation
- **Wrong technique** — a scaffolding choice is mismatched (e.g., CoT on o3)
- **Missing constraint** — an edge case was not bounded
- **Missing context** — model lacked info that only the user had
- **Reasoning directive mismatch** — think-step-by-step applied to wrong task type

Output the rewritten prompt plus a changelog:

```
VERSION: [N+1]
ROOT CAUSE: [one category above]
CHANGES:
- [section]: [specific edit]
- [rationale grounded in the observed failure]
VALIDATION TEST: [concrete input the user can run to verify the fix]
```

---

### Q6 — Multi-Vector

```
Generate 3 contrasting approaches. Label each clearly.

1. [LABEL A — e.g., PRAGMATIC]: maximum result with minimum resources.
2. [LABEL B — e.g., CONTRARIAN]: challenges the central assumption of the problem.
3. [LABEL C — e.g., USER-CENTRIC]: prioritizes the end recipient's experience.

For each approach, output:
- Objective
- Method
- Pros
- Cons
- Ideal use condition

Do not pick a winner. The user will choose.
```

This is one model listing options, not a simulated panel.

---

### Q7 — Grounding anchor

```
Use only information you are highly confident is accurate.
If uncertain about a claim, write [uncertain] immediately after it.
Do not fabricate citations, statistics, quotes, or specific dates.
If a question cannot be answered from confident knowledge, say so explicitly
instead of guessing.
```

---

### Q8 — Agentic scaffold

```
STARTING STATE: [current project / file / system state]
TARGET STATE: [specific deliverable with concrete signal of done]

ALLOWED ACTIONS:
- [explicit list]

FORBIDDEN ACTIONS:
- [explicit list — especially destructive ones]

SCOPE LOCK: only touch [files / directories / modules]. Leave [list] untouched.

STOP CONDITIONS:
- [completion criterion]
- [failure criterion — when to halt and report]

CHECKPOINT OUTPUT: after each step, output "✅ [what was completed]".

HUMAN REVIEW TRIGGERS: stop and ask before:
- deleting any file
- adding any dependency
- modifying database schema
- [other irreversible or high-risk actions]
```

---

## Why the excluded techniques are excluded

These appear in prompt-engineering writeups but fail in single-prompt execution. Do not add them under any alternate name.

| Technique | Why it fails |
|---|---|
| Mixture of Experts (prompt-embedded) | Model role-plays personas in one forward pass. No real routing. Produces hallucinated "expert consensus". |
| Meta-Prompting with simulated experts | Same failure mode as MoE. "Convene experts A, B, C" is text generation, not orchestration. |
| Tree of Thought | Model generates linear text while claiming to branch. No actual parallel exploration. |
| Graph of Thought | Requires an external graph engine. In a single prompt it is pure fabrication. |
| Universal Self-Consistency | Requires independent samples. In one context, later paths contaminate earlier ones. |
| Prompt Chaining as embedded instruction | "Execute phase 1, wait for confirmation, then phase 2" in one prompt pushes the model to fabricate intermediate confirmations. Real chaining happens at the application layer, not inside a prompt. |

If a user explicitly asks for one of these, explain briefly why it will not work as expected in single-prompt form and offer the closest working alternative (usually Multi-Vector for MoE/ToT, or advising real chaining at the application level for Prompt Chaining).
