# cfo-talks – Claude plugin

This plugin connects Claude to [CFO Talks](https://cfotalks.eu). It contains one MCP connector and one skill.

## What the connector gives

Claude gets the data of the CFO Talks orgs that you are a member of, can create financial plans in them, and can open a plan in the plan view. Claude does not see other orgs.

<!-- One row for each tool of the MCP server and of the plan view. When a tool is added, renamed or removed, change this table. -->

| Tool | What it gives |
|---|---|
| `list_orgs` | Your orgs: ID, slug and name. The other tools take the org ID. |
| `list_synthetic_accounts` | The synthetic accounts of the org's chart of accounts: code, name, category and type. |
| `list_analytic_accounts` | The analytic accounts under one synthetic account: code, name, category and type. |
| `list_plans` | The financial plans of the org, newest first: ID, name and creation time. |
| `create_plan` | Creates a financial plan in the org for a period of years, quarters, months or ISO weeks. The plan gets 1 sheet with the period in its first rows. |
| `list_plan_sheets` | The sheets of one plan, in tab order: ID and name. |
| `get_plan_sheet` | The non-empty cells of one sheet, by A1 address: the value, or the formula and its last calculated value. |
| `open_plan` | Opens one plan in the plan view: a spreadsheet in the conversation or in the side panel. |
| `query_journal` | The net turnover of accounts from the journal entries, for each year, quarter, month or ISO week of a period. A synthetic account includes the entries of all its analytic accounts. |
| `edit_plan_sheet` | A tool of the plan view. Writes values and formulas into the cells of the open plan. Claude gets it with your next message after `open_plan`, while the view is open. |
| `add_plan_sheet` | A tool of the plan view. Adds an empty sheet to the open plan, with a name and a place in the tab row. Claude gets it with your next message after `open_plan`, while the view is open. |
| `get_plan_snapshot` | Only the plan view calls it. Loads the plan into the view. |
| `save_plan_snapshot` | Only the plan view calls it. Saves the edits of the view. |

You do not call the tools yourself. Ask Claude a question, for example "Which financial plans does my org have?", and Claude calls the tools that it needs.

## The skill

The skill `cfo` tells Claude how to use the connector:

- Claude answers only from the data of the tools. It does not search the internet and does not make up numbers.
- If you do not name the org, Claude asks for it and offers your orgs.
- Before Claude creates a plan, it asks for the granularity (yearly, quarterly, monthly or weekly) and then for the first and the last year, quarter, month or week.
- Before Claude queries the journal, it asks for the granularity and then for the year, quarter, month or week.
- Before Claude edits a plan, it opens the plan in the plan view. It edits from your next message.

## The plan view

`open_plan` opens the plan view. The connector serves it as the MCP resource `ui://cfo-talks/plan-view`.

| Where | The view |
|---|---|
| claude.ai web, desktop app | In the conversation. Claude can move it to the side panel. |
| Mobile | In the conversation only. |
| Claude Code | No view. Claude gets only the ID and the name of the plan. |

The view loads the plan and saves your edits. While the view is open, Claude can change the plan with the tools of the view: `edit_plan_sheet` writes values and formulas into cells, and `add_plan_sheet` adds an empty sheet. Claude asks you to approve each call, unless you select **Always allow** for that tool. Only claude.ai web and the desktop app have the tools, and only if your organization has App tools.

If Claude opens the same plan more than once, an older card can show old numbers. Open the newest card, or reload the conversation.

## Installation

### claude.ai and the Claude desktop app

1. Open **Customize → Plugins → Add → Add marketplace**.
2. Enter `fedorocko/cfo-talks-plugin` and add the plugin **cfo-talks**.
3. Connect the connector (see [Connection](#connection)).

### Claude Code

```
/plugin marketplace add fedorocko/cfo-talks-plugin
/plugin install cfo-talks@cfo-talks-plugin
```

Then connect the connector (see [Connection](#connection)).

## Connection

The connector signs you in to CFO Talks with OAuth. The OAuth server is Clerk, the sign-in service of CFO Talks. You need a CFO Talks account. Sign-up is by invitation only.

1. Start the sign-in:
   - **claude.ai and the desktop app:** in the plugin, open the **Connectors** tab and click **Connect**.
   - **Claude Code:** run `/mcp`, select the **cfo-talks** server, then select **Authenticate**.
2. The CFO Talks sign-in page opens in your browser. Enter your email address, then the code that you get by email.
3. Allow Claude access to your CFO Talks account.

The connector is ready when its status is **Connected**.
