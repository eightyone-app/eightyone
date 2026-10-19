# Triage downtime

Goal: find where production time is lost, and why, and turn it into something someone can act on. Never invent a cause.

Read `guide_get` topic `downtime` first.

## 1. The picture

1. **`downtime_events_summary`** for the window the user cares about (default: the last 30 days): stopped time per machine, by reason.
2. **`analytics_downtime`:** time spent in a DOWN or MAINTENANCE status. It complements the stops: a machine can sit idle without a recorded stop.
3. **`analytics_oee`** for the machines that matter: is the loss in availability (stops), performance (slow running) or quality (scrap)?

## 2. Read it carefully

- **Unclassified stops** are a data gap. Report how much time they hold. Don't guess their reason.
- **Open stops** count up to now. A stop left open by mistake inflates the totals: check `downtime_events_list` with `openOnly`.
- **Planned stops** (changeovers, planned maintenance) are stops too. Separate them from unplanned ones.
- **One machine with many short stops** and **one long stop** are different problems. Say which it is.

## 3. Go deeper on the worst

For the top one or two machines:

- `downtime_events_list` for that machine: when, how long, which reasons, and whether it's recurring.
- `storylines_ssu`: what happened around the stops (tasks, status changes, maintenance).
- `analytics_reliability` (MTBF and MTTR) and `analytics_failure_pareto`: is it one failure mode repeating?

## 4. Report

Per machine:
- hours lost;
- the main reasons, with the share that's unclassified;
- whether it's recurring;
- one concrete next step, such as classifying the open stops, scheduling maintenance, investigating a failure mode, or adding a usage-based PM ([set-up-preventive-maintenance.md](set-up-preventive-maintenance.md)).

## Changing things (only with the user)

- **Classifying stops:** `downtime_events_classify`, only with the reason an operator gives you.
- **A machine without telemetry stopped or running again:** `downtime_events_open` / `downtime_events_close`.
- **Opening a repair:** `tasks_create` with a maintenance type, or `ncrs_open` if it's a quality problem.

Preview each change and get the user's OK first.
