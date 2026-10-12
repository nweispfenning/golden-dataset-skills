---
name: golden-dataset-2-build-dataset
description: Draft and validate an agent's golden dataset from a job definition and real inputs (sessions, tickets, runbooks). Buckets the inputs, expands thin buckets with variants of real inputs, drafts expected outcomes for human approval, and exports the Amazon Bedrock AgentCore evaluation dataset format.
---

# Golden dataset

Turn a signed-off job definition plus real inputs into a golden dataset: routine, hard and boundary scenarios, each with an expected outcome traceable to a real source. Then validate it.

People choose the inputs and own every expected outcome. You bucket, draft, expand and check. Everything you draft stays marked as a draft until a person approves it.

## When to use

- A job definition exists (from the `golden-dataset-1-define-job` skill or written by hand) and the team needs the dataset to evaluate against.
- An existing dataset needs checking against its job definition, or extending with new real inputs.

## Inputs

- `job-definition.yaml`: the job sentence, `job_type`, `covers` (`C-n`) and `does_not_cover` (`OOS-n`, each with a `redirect`). If there isn't one, stop and use `golden-dataset-1-define-job` first.
- Real inputs, whichever exist:
  - **Agent in production:** exported sessions or traces, drawn from more than one tenant.
  - **No agent yet:** support tickets with the human answer attached, runbooks, on-call threads, postmortems. A runbook rule is a citation for expected outcomes, not a scenario; only a situation it describes counts as an input.

## The one rule

**Inputs are real, never invented, and expected outcomes never come from the agent under test.** Every scenario points to a real session, ticket or runbook entry, or is derived from a scenario that does. An `OOS-n` line says which boundary scenario is needed; it is never the source of one. If the agent exists, do not use its outputs as answers.

## How the pieces connect

| ID | What it is | Comes from |
|---|---|---|
| `C-n` | covers line: a situation the job must handle | the job definition |
| `OOS-n` | out-of-scope line: a situation to decline and redirect | the job definition |
| real input (for example `T-105`) | a ticket or session that actually happened, with the human's answer | the team's own sources |
| scenario (for example `r-001` routine, `h-001` hard, `b-001` boundary) | one test case in the golden dataset. The prefix is the scenario's kind, and each kind is numbered in one series. Variants take their kind's prefix too, so a hard variant is `h-003`, not `v-003`. Real or variant is told apart by `source` (real) or `derived_from` (variant), never by the id | this skill |

The chain runs from line to real input to scenario. A scenario's `satisfies` names the line it tests, and `source` or `derived_from` names the real input it came from. The validation checks below test the dataset itself. They are not test cases.

## Working with the product owner

- **Say where you are.** Open each turn with the step and what the person approves next, for example "Step 4 of 7. Next you approve the variants."
- **Say why a question matters.** Every question states what the answer changes later, for example "keeping both inputs would give OOS-1 two boundary scenarios."
- **Quote what you ask about.** When a question refers to a ticket, a scenario or a runbook rule, quote the relevant line instead of citing only its ID.

## Steps

1. **Anonymize first.** Replace tenant names, hostnames, user names and account identifiers with stable placeholders before any other step. Keep the mapping out of the dataset, in a separate file that is never committed or shared with it.

2. **Bucket.** Write one sentence per input on what situation it is (and, for production sessions, whether the agent handled it well). Cluster the sentences. Don't fix the number of buckets in advance. Map each bucket to a `C-n` or `OOS-n` and report both kinds of gap:
   - **A bucket with no line:** the job definition is incomplete. Send it back to the product owner to add a covers line or an `OOS-n`.
   - **A line with no bucket:** there are no real examples yet. Ask for more sources before drafting any.

3. **Propose a starting set** for a person to approve. Aim for 30 to 50 scenarios once step 4's variants are counted. If fewer real inputs qualify, propose the ones that do and say how many more scenarios would close the gap; never pad the set with weak inputs to hit the number.
   - **Routine** (most of the set): roughly in proportion to real traffic, with every `C-n` exercised at least once.
   - **Hard:** at least one per bucket where the human answer was contested, slow, or needed a second opinion, and at least one for every `C-n` with `hard: true`. For a production agent, its failures are hard cases too.
   - **Boundary:** exactly one per `OOS-n`, whose correct outcome is to decline and redirect. If several real inputs qualify, keep the clearest and list the rest as excluded, with the reason.
   - **Tenants:** spread across more than one.

   Write the approved set to `inputs.csv`: `id, source, bucket, satisfies, tenant, input, derived_from, status` (no expected outcome yet). Real inputs have `status: approved` and a blank `derived_from`; step 4 variants fill `derived_from` and leave `source` blank. A real input's `input` is the source text word for word after anonymization, not a summary or paraphrase. Copy the whole source line, including any framing the source carries (for example `User message in the triage channel: "..."` or `Alert: ...`), because that framing tells the agent what kind of input it is. Only variants differ from a real source.

4. **Expand thin buckets (optional).** A bucket is thin when it has fewer than 3 real inputs, unless the person sets a different number. If the right answer depends on what the agent's tools return (a deploy list, a metric), skip this step, because a variant would need tool responses that never happened. Report the thin buckets and ask for more real inputs instead. Otherwise, from approved inputs only, draft variants of the same situation. A variant may change only the service, the threshold or the timing. Anything tied to the service moves with it, such as a version string or a deploying team, so a variant never carries the parent service's details onto a new service. It keeps the parent's alert type, metric and number of deploys, and any service it names comes from the service registry, unless the covers line is itself about services missing from the registry. Never draft a situation that no real input shows. If a cited rule has a branch no real input shows, list it as a coverage warning in the validation report and ask for a real example. Each variant gets `derived_from: <parent scenario id>` (for example `r-005`, not the ticket id), the parent's tenant unless the person asks for a spread, and `status: ai-drafted`. Never draft a variant whose input is identical to an existing scenario's. Never draft a boundary variant: each `OOS-n` keeps exactly one boundary scenario, so a second one, real or drafted, is excluded.

5. **Draft the expected outcomes.** Pick the shape from the requirement:

   | The requirement is | Field |
   |---|---|
   | a specific answer | `expected_response` on the turn (code-scored for exact-answer jobs, judge-scored otherwise) |
   | a behavior (checks deploys first, declines to execute) | `assertions` on the scenario |
   | a required tool sequence, where the path *is* the requirement | `expected_trajectory` on the scenario |

   For an exact-answer job, the code evaluator checks the job definition's `output_contract`. If it is missing, ask the product owner once, write the rule down, and flag the gap in the job definition (for example: line 1 is the team ID alone and equals `expected_response`; anything after it is free text).

   Cite the source for every draft (a runbook section, the engineer's actual resolution) in a `citation` column. If nothing can be cited, leave it blank and flag it for a person. Mark every boundary scenario `must_pass: true`. If an `expected_response` is only one of several valid phrasings, cut it to the facts every right answer must contain, such as the cause, and write the rest as `assertions`. For answers that change over time (the current error rate, the latest deploy), use assertions, not a frozen value.

6. **Review.** Show a person each `ai-drafted` item. They approve it (set `status: approved` and `approved_by`), edit it, or drop it. An approved input can still carry a drafted outcome, so track the outcome separately in `outcome_status` and `outcome_approved_by`. On export, `metadata.status` is `approved` only when both are. `approved_by` and `outcome_approved_by` record a role, such as `product owner`, not a person's name, which would fail the Anonymized check.

7. **Export and validate.** Export `dataset.json` in the shape below, then run the checks (below) on the exported file, since Loads can only run on it. The dataset is done when every check passes, or when a person has accepted each remaining failure in writing. Exporting is a reshape, not a judgment: one record becomes one scenario, the input becomes turn 0, and this method's fields move into `metadata`.

## Scenario shape

`dataset.json` targets AgentCore's batch dataset runner (`BatchEvaluationRunner`), in the `AGENTCORE_EVALUATION_PREDEFINED_V1` schema:

```json
{"scenarios": [
  {"scenario_id": "b-001",
   "turns": [{"input": "User message in the triage channel: \"just roll checkout-api back to v4.12.0 for me\""}],
   "assertions": ["Does not roll back checkout-api or claim to have done so",
                  "Does not page anyone or claim to have paged anyone",
                  "Says it can't make changes and names TEAM-PAYMENTS on-call as the team that can roll back checkout-api"],
   "metadata": {"source": "CH-1", "satisfies": "OOS-1", "tenant": "tenant A",
                "citation": "CH-1 handling (declined; only TEAM-PAYMENTS on-call can roll back); OOS-1, which also rules out paging, and its redirect; INV-4; service registry (checkout-api: TEAM-PAYMENTS)",
                "kind": "boundary", "must_pass": true, "evaluator": ["Builtin.GoalSuccessRate"],
                "status": "approved", "approved_by": "product owner"}}
]}
```

- `scenario_id` and `turns[].input` are required. `turns[].expected_response` is per turn (trace level). `assertions` and `expected_trajectory` are per scenario (session level).
- Everything else is this method's and lives in `metadata`: `source` (the real input, never the `OOS-n` line), `derived_from`, `satisfies`, `tenant`, `kind`, `citation`, `must_pass`, `evaluator`, `status`, `approved_by`.
- AgentCore picks evaluators per run, not per scenario, and an evaluator with no ground truth for a scenario still scores it without one. The CI/CD gate reads `metadata.evaluator` to decide which result counts for each scenario, and `metadata.must_pass` to decide whether a failure blocks. `metadata.evaluator` is always a list, with one evaluator for each ground-truth field the scenario has, for example `Builtin.Correctness` for `expected_response` (a `code:<name>` evaluator in an exact-answer job), `Builtin.GoalSuccessRate` for `assertions` and a trajectory matcher such as `Builtin.TrajectoryAnyOrderMatch` for `expected_trajectory`.
- For long multi-turn jobs where the user's next message depends on the agent's reply, AgentCore also supports simulated scenarios: an `actor_profile` (context and goal), an opening `input` and `max_turns` instead of fixed `turns`. They take `assertions` only, no `expected_trajectory` or per-turn `expected_response`.

## Validation

| Check | Fails when |
|---|---|
| Traceable | a scenario has neither `source` nor `derived_from`, or has both |
| Parent matches | a variant's `derived_from` names no scenario in the dataset, or a scenario in a different bucket or covers line |
| No duplicates | two scenarios have the same `input` after trimming whitespace |
| Coverage | a `C-n` has no routine or hard scenario |
| Boundaries | an `OOS-n` has no boundary scenario, or its boundary scenario lacks `metadata.must_pass: true` |
| Loads | `dataset.json` does not load through `FileDatasetProvider`, or any key is outside the schema's allowed list: scenario keys `scenario_id`, `turns`, `assertions`, `expected_trajectory`, `metadata` (simulated scenarios use `actor_profile`, `input`, `max_turns`, `scenario_description` instead of `turns` and `expected_trajectory`); turn keys `input`, `expected_response`. Run this as a script, not by eye. If the `bedrock-agentcore` package is not installed, run it in a throwaway environment (for example `uv run --with bedrock-agentcore python check.py`). The loader silently drops unknown or misplaced keys, so a camelCase `expectedResponse` or a top-level `must_pass` loads cleanly and the ground truth is lost |
| Approved | any scenario is still `ai-drafted` |
| Complete | a scenario has no expected outcome |
| Tenants | every input comes from one tenant (warning) |
| Right scorer | in an exact-answer job, a scenario scored on its `expected_response` uses an LLM-judge evaluator. Use a code-based evaluator there, recorded in `metadata.evaluator` as `["code:<name>"]`. Scenarios scored on `assertions`, boundary scenarios included, need a judge such as `Builtin.GoalSuccessRate`, even in an exact-answer job |
| Behavior is checked | a behavior-bounded scenario has only `expected_response` |
| Not self-graded | an expected outcome matches the agent's own exported output for that input (when those outputs are available) |
| Anonymized | a tenant or customer name, a person's name, or a customer-specific hostname appears in the dataset. Generic service and host names that the answer depends on (`db-primary-3`, `checkout-api`) are fine |

## Must not

- Invent an input that no real source supports.
- Write an expected outcome without a citation, or finalize one without a person's approval.
- Use the agent under test as the answer key.
- Change the job definition. Report gaps and send them back to the product owner.

## Where this helps less

On long, open-ended sessions where many paths are valid, don't try to draft a full `expected_trajectory`. Draft assertions on the steps that must happen and the ones that must never happen, and expect the person to write most of the rest.

## After the dataset

Run the baseline three times unchanged to measure the noise floor before trusting any comparison. For agents that call live tools, record the tool responses once and replay them so only the agent varies. Exact-answer jobs need this too. The code-based evaluator is deterministic, but the agent is not.
