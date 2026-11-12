---
name: ai-ambition
description: Turn an ordinary AI request into a more ambitious one along three axes - Altitude (aim at the outcome, not the task), Depth (work the way a top practitioner would, with an outside judge) and Range (bring several fields to bear) - distilled from Felix Rieseberg's "How to be more ambitious - a tutorial for humans with models". Use when the user asks for a bigger or better version of a task, or when a request is visibly one small step of a larger goal (fix one bug, write one report, build one deck, answer one question) and they want more. Triggers - be more ambitious, think bigger, what's the bigger version, level this up, 更有野心, 往大了想, 升级一下这个任务, 能不能做得更狠, 不要只修这一个. Skip for mechanical work that only needs to be faster and for requests that explicitly want a small change.
---

# AI ambition: make the task bigger, deeper, wider

Source: Felix Rieseberg, "How to be more ambitious: A tutorial for humans
with models". This file is a paraphrased method, not a copy of the essay.

Premise: model capability keeps growing, unevenly, while most requests stay
at "answer a question / do a small task". The limit is usually the asker's
belief about what is possible, not the model. Specific, hard goals beat
"do your best" (Locke and Latham's review of goal-setting research).

## The four kinds of ambition

| Axis | One line | Signal |
|---|---|---|
| Speed | Same work, faster | The default. Needs no skill; ignore it here. |
| **Altitude** | Own the outcome instead of the task | "fix this crash" -> "keep the app crash-free" |
| **Depth** | Do it the way the best practitioners do, until an expert could learn from the result | markdown table -> real database and statistical model |
| **Range** | Look at the work through several fields at once | one author -> analyst + journalist + editor + red team |

## Procedure

Reply in the user's language. Do steps 1-5 on paper first, present the
result in the format of step 6, and only start work after the user picks.

### 1. Check fit and name the goal behind the goal

- Good fit: goal-driven work with a concrete outcome (a metric, reliability,
  a decision, a diagnosis). Strong signal: the task would normally take
  meetings to move forward.
- Weak fit: creative or taste-driven work. Skip Altitude there; Range still
  helps.
- Too vague also fails: a bare "make money"-style goal does not work on its
  own. Raise the altitude only as far as the outcome stays concrete.
- State in one sentence what should be true once this is done, beyond the
  thing that was literally asked for.

### 2. Altitude: offer a three-rung ladder

Rewrite the same task at three rising levels and let the user choose.

1. **Analysis** — hand over all the data available and ask what changed,
   why, and what to do about it.
2. **Decisions** — find the three most urgent decisions and draft the case
   for and against each.
3. **System** — design a better mechanism than the current artifact (for a
   quarterly review: a way to catch problems earlier than a quarterly deck),
   backtest it against history, then run it alongside the existing process
   for one cycle.

Example: "fix this crash" -> "find the shared root cause of this class of
crash" -> "keep the app crash-free: detect, reproduce, fix and verify
without being asked".

### 3. List what sustained work needs

The system rung only works if the model can keep going without a human in
every loop. List each item as *have* or *must build*, and build the missing
ones first:

- **Raw data access** — crash reports, logs, databases, metrics, fetched
  directly rather than pasted in.
- **Mappings** — release <-> commit, event <-> code, metric <-> owner.
- **A place to reproduce and verify** — real environments, every platform
  that matters.
- **A machine the model can use** — one where it may install software.
- **A trigger** — a schedule or loop, if the work should recur.

### 4. Depth: rewrite to the expert standard

Deeper is not bigger: keep the scope, move closer to raw data, real tools
and real methods. Before rewriting the prompt, answer four questions and put
the answers into it:

1. What would a top professional in this field do that was not asked for?
2. What is the rawest material they would start from, instead of a summary?
3. Which specific methods and tools would they use?
4. What, other than the user or the model, will show the result is right?

Then add whichever of these apply:

- **Real instruments** — databases, statistical models, profilers,
  simulations. Ask which methods experts use if unsure; the model usually
  knows.
- **An unreasonable target, stated as reasonable** — need 10s down to 5s?
  Ask for 200ms. Push again when the first answer is "that is not feasible".
- **Attack the assumptions** — test what everyone believes against raw data
  ("is churn really an onboarding problem?").
- **Produce new knowledge** — build a model or simulation from logs, look
  for the earliest signal nobody tracks, check predictions on unseen data.
- **An outside judge** — never let the model grade its own work. Prefer a
  check the model can run itself and get immediate feedback from: tests,
  benchmarks, backtests, probes against a real environment.

### 5. Range: assemble a crew

- **Swap the noun for a verb** (from *The Art of Innovation*). Rename the
  deliverable by what it actually does: "quarterly review" -> "get busy
  people to believe a cause-and-effect story and decide under uncertainty in
  30 minutes". The verb shows which skills are needed.
- **List the crew** the verb calls for: analyst, journalist, editor,
  designer, plus a red team (short seller, forensic accountant).
- **Do the job, do not role-play it.** Ask the model to do what these people
  do, not to write in their voice. Subagents work well: one perspective
  each, then one synthesis.
- **Invite one unexpected tool or field.** The surprising results tend to
  come from a field the user would not have reached for.

### 6. Output format

```
Goal behind the goal: <one sentence>
Ladder (pick one):
  A Analysis:  ...
  B Decisions: ...
  C System:    ...
Prerequisites: <have / must build>
Depth: <unreasonable target, raw material, tools, outside judge>
Range: <verb rewrite> -> <crew>
Recommendation: <rung + one-line reason>
Ready-to-use prompt: <full prompt for the recommended rung>
```

Then ask which rung to take, and start only after the answer.

## Boundaries

- Ambition raises **goals and standards**, never **permissions**. Anything
  irreversible, outward-facing or large still needs the same confirmation it
  needed before.
- A system-rung plan that creates schedules, writes to external systems or
  touches production is listed for approval first, not executed by default.
- Every unreasonable target comes with a stated acceptable fallback, so
  missing it does not tempt the model into faking results or weakening the
  tests.
- The outside judge must really run. If none exists, say plainly that the
  step cannot be verified objectively.
