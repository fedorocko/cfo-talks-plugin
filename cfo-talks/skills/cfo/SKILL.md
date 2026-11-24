---
name: cfo
description: "Work with CFO Talks data using ONLY the cfo-talks MCP connector: orgs, chart of accounts, journal turnover and financial plans. Use when the user asks about their accounts, turnover or journal, or wants to list, read, create, open or edit a financial plan."
metadata:
  version: "1.3.0"
---

# cfo-talks:cfo

Answer from the cfo-talks MCP connector only. Its tools: `list_orgs`, `list_synthetic_accounts`, `list_analytic_accounts`, `query_journal`, `list_plans`, `create_plan`, `list_plan_sheets`, `get_plan_sheet`, `open_plan`. While the plan view is open, the view adds `edit_plan_sheet` and `add_plan_sheet`.

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

## Edit a plan

1. Know the org (rule 2) and the plan. If more than one plan fits the request, ask which plan the user means.
2. If the view of this plan is already open, do not call `open_plan` again: an older card of the plan can then show old numbers. Else call `open_plan`. `edit_plan_sheet` and `add_plan_sheet` are tools of the plan view: you get them from the user's next message, while the view is open. If you called `open_plan` in this message, ask the user to send the edit in the next message.
3. Get the sheet ID from `list_plan_sheets`, and the addresses from `get_plan_sheet`. Never guess an address: find the row and the period column in the cells.
4. Write only what the user asked for. Put all edits of one request into one `edit_plan_sheet` call: Ctrl+Z undoes one call.
5. Report the values of the result. A formula has its calculated value there. If you opened this plan more than once in this conversation, tell the user to check the newest card.
6. If you have no `edit_plan_sheet` tool, say that Claude can edit a plan only with the plan view open, and only if the organization has App tools.

## Add a sheet to a plan

1. Do steps 1 and 2 of **Edit a plan**.
2. Call `add_plan_sheet` with the name that the user gave. Send `position` only when the user said where the sheet goes: 1 is the first tab, as the order of `list_plan_sheets`. Without `position`, the sheet goes after the last sheet.
3. The new sheet is empty. To fill it, call `edit_plan_sheet` in the same message, with the `id` of the result as `sheetId`.
4. Report the name and the position of the result. If the name of the result is not the name that you sent, the plan already had a sheet with that name: tell the user the real name.
5. If you have no `add_plan_sheet` tool, step 6 of **Edit a plan** applies.

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
