# Glossary

The IDs, fields and evaluators that the two skills, the sample and the blog post use, and how they connect.

## IDs

A dataset has two kinds of ID. The skills assign the first kind the same way every time. The second kind comes from your own sources, so it changes with your data.

**Assigned by the skills**

| ID | What it is | Assigned by |
|---|---|---|
| `C-1`, `C-2`, … | A "covers" line in the job definition, a situation the job must handle | Skill 1, the job-definition skill |
| `OOS-1`, `OOS-2`, … | A "does not cover" (out-of-scope) line, a request the agent should decline and redirect | Skill 1 |
| `r-001`, `h-001`, `b-001`, … | A scenario. The letter is its kind, `r` for routine, `h` for hard and `b` for boundary. Variants use the same letters, and `derived_from` marks them | Skill 2, the dataset skill |

**From your sources**

Each scenario's `source` and `citation` keep the IDs your sources already use. In the sample, those are:

| ID | What it is | Sample file |
|---|---|---|
| `PM-101` to `PM-114` | a postmortem | `examples/incident-analysis/sources/postmortems.md` |
| `CH-1` to `CH-4` | a request from the triage channel | `examples/incident-analysis/sources/triage-channel.md` |
| `INV-1` to `INV-4` | an investigation rule in the runbooks | `examples/incident-analysis/sources/runbooks.md` |

Your dataset will have its own instead, such as ticket numbers, session IDs or runbook section names.

## From job type to metric

The job definition's `Type` decides what a right answer has to show. Each of those things is an expected-outcome field, each field has an AgentCore evaluator that grades it, and each evaluator's pass rate across the scenarios it counts for is a metric.

| Type | A right answer has to show | Expected-outcome fields | Evaluators |
|---|---|---|---|
| Exact-answer | the one right value, such as a team ID | `expected_response`, compared exactly | a code-based evaluator, which is your own Lambda function |
| Judged-answer | the right content, in any of many phrasings | `expected_response` or `assertions` | `Builtin.Correctness` or `Builtin.GoalSuccessRate`, both LLM judges |
| Behavior-bounded | the facts, the behavior and, where the path matters, the right lookups | `expected_response`, `assertions` and `expected_trajectory` | `Builtin.Correctness`, `Builtin.GoalSuccessRate` and a trajectory matcher such as `Builtin.TrajectoryAnyOrderMatch` |

Boundary scenarios check behavior only, so in every type they get `assertions`, graded by `Builtin.GoalSuccessRate`, and `must_pass: true`.

The sample's job, incident analysis, is behavior-bounded. It tracks three metrics: whether the agent named the right cause (Correctness), did what it must (GoalSuccessRate) and made the right lookups (TrajectoryAnyOrderMatch).

## Expected-outcome fields

AgentCore calls expected outcomes ground truth. The field names are AgentCore's.

| Field | Holds | Graded on |
|---|---|---|
| `expected_response` | the facts a right reply must contain | each reply |
| `assertions` | statements the whole session must satisfy, such as "takes no action" | the whole session |
| `expected_trajectory` | the tool calls the agent should make | the session's tool calls |

An evaluator that runs on a scenario without its field uses its own judgment instead, which is why `metadata.evaluator` lists only the evaluators whose fields a scenario has. Evaluators that never use a field, such as Helpfulness, Faithfulness and the tool-call evaluators, can be added to any run.

## Metadata

Everything in a scenario's `metadata` belongs to this method. AgentCore's dataset format allows it but doesn't use it for scoring, and a CI check can read it.

| Key | Meaning |
|---|---|
| `source` | the real input the scenario came from, by your source's own ID |
| `derived_from` | for a variant, the scenario it was drafted from |
| `satisfies` | the `C-n` or `OOS-n` line the scenario tests |
| `tenant` | the tenant the input came from, after anonymization |
| `citation` | where the expected outcome came from, such as a runbook rule or a postmortem's confirmed cause |
| `kind` | `routine`, `hard` or `boundary` |
| `must_pass` | `true` for boundary scenarios, so a single failure blocks the change |
| `evaluator` | the evaluators whose results count for the scenario, one per expected-outcome field it has |
| `status` and `approved_by` | `approved` once a person has approved the scenario and its expected outcome, and that person's role |
