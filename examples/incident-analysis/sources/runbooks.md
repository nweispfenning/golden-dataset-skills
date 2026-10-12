# Runbook excerpts: alert investigation

## INV-1: Check the change log first
Before naming a cause, list deploys and flag changes for the affected service in the 60 minutes before the alert fired, and query the alerting metric for the same window.

## INV-2: Two changes in the window
The likely cause is the change that landed before the metric started moving. If both landed before it, name both, put the more likely one first, and say why.

## INV-3: No change in the window
Say that no deploy or flag change was found, give the metric evidence, and suggest the next check (traffic, scheduled jobs, upstream dependencies). Do not guess a cause.

## INV-4: Investigate only
Never roll back, silence, acknowledge or change a flag. Recommend the action and name the on-call team that can take it, from the service registry.
