---
name: golden-dataset-1-define-job
description: Interview a product owner and draft a checked job definition (one-sentence job, coverage lines, out-of-scope lines) for building an agent's golden dataset.
---

# Golden-dataset job definition

Produce the **job definition** an agent's golden dataset is built from: one sentence stating the job, the situations it covers, and the situations it does not, each with an id. Then check it before any data is gathered.

The product owner decides everything that goes into it. You ask the questions, write the page, and point out problems. You never decide scope yourself.

## When to use

- A team is about to build or evaluate an agent and has no written definition of what it is responsible for.
- An existing agent's scope has drifted, or its eval results are hard to interpret because nobody agrees what "correct" means.
- Before the `golden-dataset-2-build-dataset` skill: that skill takes this skill's output as its input.

Works the same whether or not the agent exists yet. The interview, the output and the checks don't change. Only step 3 differs: for an existing agent you compare the answers against its prompt and tools; for a new one, against the planned design and the manual process it replaces.

## Vocabulary

- **Agent**: the software under test.
- **Job**: one bounded responsibility, stated in one sentence. One golden dataset per job. One agent may do several jobs.
- **Covers line** (`C-n`): a situation the job must handle. Each becomes a coverage requirement for the dataset.
- **Out-of-scope line** (`OOS-n`): a situation the job must decline, and where to send the user instead. Each becomes one boundary scenario in the dataset.
- **Job type**: exact-answer (one checkable answer), judged-answer (one right answer, many phrasings), or behavior-bounded (what the agent *does* matters as much as what it says).

## How the pieces connect

| ID | What it is | Comes from |
|---|---|---|
| `C-n` | covers line: a situation the job must handle | this skill, approved by the product owner |
| `OOS-n` | out-of-scope line: a situation to decline and redirect | this skill, approved by the product owner |
| real input (for example `T-105`) | a ticket or session that actually happened, with the human's answer | the team's own sources |
| scenario (for example `r-001`, `h-001`, `b-001`) | one test case in the golden dataset | the `golden-dataset-2-build-dataset` skill |

The chain runs from line to real input to scenario. Each scenario names the line it tests and the real input it came from. The checks below test the job definition itself. They are not test cases.

## Working with the product owner

- **Say where you are.** Open each turn with the step and what the person approves next, for example "Step 2 of 6. Next you approve the job sentence."
- **Say why a question matters.** Every question states what the answer changes later, for example "this decides whether T-110 is scored by code or by a judge."
- **Quote what you ask about.** When a question refers to a ticket or a runbook rule, quote the relevant line instead of citing only its ID.

## Steps

1. **Collect what exists.** Ask for: the agent's system prompt and tool list if it exists; a PRD or feature brief; runbooks; three real cases that went well and one that went badly. Ask for artifacts, not opinions.

2. **Interview, one question at a time.** Stop when each question has a concrete answer.
   - In one sentence, what is the job? What starts it (a user message, an alert, a schedule)?
   - What inputs does it receive? Across which customers or tenants?
   - What situations are hard today, and what makes them hard?
   - What must it never do? For each: where should the user go instead?
   - When the input is ambiguous, what is the right behavior: best guess with a reason, ask, or escalate?
   - What does a good outcome look like: a label, a piece of text, or a sequence of actions? For an exact-answer job, also ask for the exact reply format (for example: "line 1 is the team ID alone; anything after it is free text"). That becomes the `output_contract`, and the dataset's code-based evaluator checks only what it names. Then ask what line 1 holds for every outcome that is not the usual answer, such as an escalation with no owning team (for example `INCIDENT-MANAGER`), and write each one into the contract.

3. **Compare the answers with what already exists.** Ask the product owner about each mismatch: in scope, out of scope, or a mistake.
   - **Agent exists:** list every capability its prompt or tools imply that the product owner did not name, and every capability they named that nothing supports. This is where scope drift shows up.
   - **No agent yet:** compare against the planned design (PRD, intended tools, draft prompt) the same way. Then compare against the manual process the agent replaces: list what the people doing the job today handle that the interview didn't mention. Each item is a missing covers line or a missing `OOS-n`.

4. **Draft** `job-definition.yaml` (schema below) and a one-page `job-definition.md` for people to read.

5. **Run the checks** (below). Report every failure plainly. The product owner fixes it, or accepts it and records why in `notes`.

6. **Get sign-off.** Put the product owner's name in `owner`. An unsigned definition is a draft.

## Output schema

The example is illustrative. Copy its structure, never its lines: every covers and out-of-scope line comes from the product owner.

```yaml
job: "Route each incoming alert to the team that owns the affected service."
job_type: exact-answer          # exact-answer | judged-answer | behavior-bounded
trigger: [alert-fired]          # one or more: user-message | alert-fired | schedule | other
output_contract: "Line 1 is the owning team ID alone. Anything after line 1 is free text and is not scored."   # required for exact-answer jobs
covers:
  - id: C-1
    text: "Alerts from any monitored service, any tenant"
  - id: C-2
    text: "Alerts with or without a recent deploy attached"
  - id: C-3
    text: "Ambiguous ownership: returns the most likely team"
    hard: true                  # where real cases show difficulty: slow, contested, or needed a second opinion
does_not_cover:
  - id: OOS-1
    text: "Changing anything: no silencing, acknowledging or rollbacks"
    redirect: "Name the on-call owner who can act"
  - id: OOS-2
    text: "Explaining root cause"
    redirect: "Point to the alert-explanation assistant"
owner: "<name, role>"           # the product owner who signed off
sources: ["<runbook>", "<ticket queue>"]
notes: []                       # accepted check failures, with reasons
```

## Checks

| Check | Fails when |
|---|---|
| One job | the sentence joins two outputs ("routes alerts and summarizes incidents") |
| Redirects | an `OOS-n` has no `redirect` |
| No contradiction | a covers line and an out-of-scope line describe the same situation |
| Type fits | the described output doesn't match `job_type` (for example, a label-producing job marked judged-answer) |
| Contract stated | an exact-answer job has no `output_contract`, or the contract does not say what line 1 holds for an outcome that is not the usual answer (an escalation, a decline) |
| Hard cases named | no covers line is marked `hard`; every real job has somewhere it is difficult. Mark the lines where real cases were slow, contested or needed a second opinion, not only lines that sound hard |
| **Split test** | the covers lines need more than one job type, or would need rulers that conflict. Report it as "this looks like N jobs," propose the split, and produce one definition per job if the owner agrees |

## Must not

- Add a covers or out-of-scope line the product owner did not approve.
- Choose the job type, or settle an ambiguous behavior, without confirmation.
- Fill a gap by guessing. Leave it blank and ask.
- Treat the existing agent's behavior as the definition of the job. The agent is what gets measured, not what defines correct.

## Example: the split test firing

Input job: "Reduce alert noise in the customer's environment."

The checks find three outputs of different types: a noise/actionable label (exact-answer), a grouping of correlated alerts (judged-answer), and a recommended suppression rule (exact-answer where it validates, judged where it does not). Report: "This is three jobs: classify, group, recommend. Each needs its own definition and dataset, with a router in front."
