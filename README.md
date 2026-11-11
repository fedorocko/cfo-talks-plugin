# cfo-talks – Claude plugin

This plugin connects Claude to [CFO Talks](https://cfotalks.eu). It contains one MCP connector and no skills.

## What the connector gives

Claude gets the data of the CFO Talks orgs that you are a member of. Claude does not see other orgs.

<!-- One row for each tool of the MCP server. When the server adds, renames or removes a tool, change this table. -->

| Tool | What it gives |
|---|---|
| `list_orgs` | Your orgs: ID, slug and name. The other tools take the org ID. |
| `list_synthetic_accounts` | The synthetic accounts of the org's chart of accounts: code, name, category and type. |
| `list_analytic_accounts` | The analytic accounts under one synthetic account: code, name, category and type. |
| `list_plans` | The financial plans of the org, newest first: ID, name and creation time. |

You do not call the tools yourself. Ask Claude a question, for example "Which financial plans does my org have?", and Claude calls the tools that it needs.

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
