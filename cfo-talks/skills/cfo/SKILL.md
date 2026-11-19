---
name: cfo
description: "Work with CFO Talks data using ONLY the cfo-talks MCP connector: orgs, chart of accounts, journal turnover and financial plans. Use when the user asks about their accounts, turnover or journal, or wants to list, read, create or open a financial plan."
metadata:
  version: "1.1.0"
---

# cfo-talks:cfo

Answer from the cfo-talks MCP connector only. Its tools: `list_orgs`, `list_synthetic_accounts`, `list_analytic_accounts`, `query_journal`, `list_plans`, `create_plan`, `list_plan_sheets`, `get_plan_sheet`, `open_plan`.

## Rules

1. **Only the connector's data.** Every number and fact in the answer comes from a cfo-talks tool result. Never search the internet, never take figures from general knowledge, and never estimate, extrapolate or make up numbers. If the tools do not return it, say so.
2. **Always know the org.** If the user did not name the org, ask which one before you do anything else. Call `list_orgs` and offer the orgs by name. Pass the org's `id` as `orgId`. Never pick an org for the user.
3. **Interview before `create_plan` and `query_journal`**, as described below. Ask one question at a time and wait for the answer. If the user already said it in the request, confirm it instead of asking again. Never assume a granularity or a period that the user did not give.

## Create a plan

1. Know the org (rule 2).
2. Ask for the granularity: yearly, quarterly, monthly or weekly.
3. Ask for the range:
   - **Yearly:** from which year to which year.
   - **Quarterly:** from which quarter of which year to which quarter of which year.
   - **Monthly:** from which month of which year to which month of which year.
   - **Weekly:** from which week of which year to which week of which year.
4. Build `period` from the answers and call `create_plan`:

   | Granularity | `period` |
   |---|---|
   | Yearly | `{ "granularity": "year", "from": { "year": 2027 }, "to": { "year": 2029 } }` |
   | Quarterly | `{ "granularity": "quarter", "from": { "year": 2027, "quarter": 1 }, "to": { "year": 2027, "quarter": 4 } }` |
   | Monthly | `{ "granularity": "month", "from": { "year": 2027, "month": 1 }, "to": { "year": 2028, "month": 6 } }` |
   | Weekly | `{ "granularity": "week", "from": { "year": 2027, "week": 1 }, "to": { "year": 2027, "week": 13 } }` |

   `quarter` is 1–4, `month` is 1–12, `week` is an ISO week. `from` and `to` are inclusive, and `from` is not after `to`.
5. Call `open_plan` with the returned `id` to show the plan.

## Query the journal

1. Know the org (rule 2).
2. Ask for the granularity: yearly, quarterly, monthly or weekly.
3. Ask for the unit:
   - **Yearly:** which year.
   - **Quarterly:** which quarter of which year.
   - **Monthly:** which month of which year.
   - **Weekly:** which week of which year.
4. Build `period` with `from` equal to `to`, for example `{ "granularity": "month", "from": { "year": 2025, "month": 3 }, "to": { "year": 2025, "month": 3 } }`.
5. Find the account codes with `list_synthetic_accounts` and `list_analytic_accounts`. Never guess a code. If it is not clear which account the user means, ask.
6. Call `query_journal` and report each account's `net` as returned, with its code and name. For the total of a synthetic account, query the synthetic account. Do not add up its analytic accounts.

## Plans and sheets

To read an existing plan: `list_plans`, then `list_plan_sheets`, then `get_plan_sheet`. To show it: `open_plan`.
