# Tredict MCP Server

Official Model Context Protocol (MCP) server for [Tredict](https://www.tredict.com), the planning and analytics platform for endurance sports athletes and coaches. Connect your preferred AI assistant to your complete training history across running, cycling, swimming and more. Analyse activities, determine capacity values, create fully structured training plans inside Tredict, synced directly to your watch.

Tredict also ships interactive **MCP App UI widgets** that visually render activity details and training plans directly in the LLM chat of hosts such as Claude.ai or ChatGPT.

## Overview

| | |
|---|---|
| **Name** | Tredict MCP Server |
| **MCP Server URL** | `https://www.tredict.com/api/mcp/v2` |
| **Transport** | Streamable HTTP and SSE |
| **Protocol** | MCP 2025-06-18 |
| **Authentication** | OAuth 2.1 with PKCE / Personal API Token |

Full technical documentation: [Tredict MCP Server Documentation](https://www.tredict.com/blog/mcp_server_docs/).

## Core features

- **Training plan creation.** Analyse training history and current fitness to generate reusable, multi-week plans with structured workouts in the user's "My Plans" category.
- **Structured workout authoring.** Create executable structured workouts for running, cycling, swimming and strength, and reschedule or rename planned sessions.
- **Activity analysis.** Detailed metrics and full time series (heart rate, pace, power, cadence, altitude, position, laps) for every executed activity.
- **Capacity assessment.** Determine FTP, FTPace (running), HRmax and lactate threshold from recent training data.
- **Health and recovery.** Access HRV, sleep, body values (weight, body fat, resting heart rate) and training effort over time.
- **Training zones.** Read zone revisions and aggregated zone distribution per sportType and zoneType (heartrate, pace, power, cadence).
- **Equipment tracking.** List all equipment (bikes, shoes, swimming, misc) with usage ranges, counts, distance and duration.
- **Device sync.** Plans applied to the training calendar are automatically uploaded to connected sports watches (Garmin, Suunto, Coros, icTrainer, Wahoo, Apple Watch and others).

## Example Conversations

### Activity Analysis

```
You: "How did my long run on Sunday go?"
Claude: [Shows pace, heart rate, cadence, elevation and zone distribution. 
         Flags the last 5 km where HR drifted into threshold]
```

### Training Plan Creation

```
You: "Build me a 10-week half marathon plan based on my recent training"
Claude: [Analyses last 8 weeks of running history, creates a structured 
         multi-week plan in My Plans, pushes workouts to your Garmin]
```

### FTP & Capacity

```
You: "What is my current running FTP?"
Claude: [Derives FTPace from recent efforts, compares to last revision, 
         suggests updated training zones]
```

### Equipment & Shoes

```
You: "How many kilometres do my current running shoes have?"
Claude: [Lists active shoes with distance logged, 
         flags gear approaching typical replacement range]
```

### Year-over-Year Comparison

```
You: "How does my 2025 training compare to 2024?"
Claude: [Compares total volume, zone distribution, long run progression
         and peak weeks across both years]
```

### Recovery Check

```
You: "Am I ready for tomorrow's threshold session?"
Claude: [Reads HRV trend, resting heart rate and recent training load.
         Gives a clear go or back-off recommendation]
```

### Plan Adherence

```
You: "Am I actually hitting my training zones as planned?"
Claude: [Shows prescribed vs actual time in each zone per sport,
         highlights where execution is drifting from the plan]
```

### HRV & Fitness Trend

```
You: "What does my HRV trend tell me about my current fitness?"
Claude: [Analyses rMSSD baseline, flags suppression periods,
         links dips to high-load weeks in training history]
```

### Race Goal Reality Check

```
You: "Is my sub-4 marathon goal realistic given my current training?"
Claude: [Derives pace capacity from recent long runs and threshold efforts,
         compares to goal pace, outlines what needs to improve]
```

## Platform support

**Officially supported:** Anthropic Claude (Web, Desktop, Code), OpenAI ChatGPT, OpenAI Codex, Mistral Le Chat, Mistral Vibe.

**Compatible:** any MCP client that supports OAuth 2.1 or bearer-token authentication, including local LLM setups.

Setup guides per platform:

- [Claude Web / Claude Desktop](https://www.tredict.com/faq/connect-claude-web-with-tredict/)
- [Claude Code](https://www.tredict.com/faq/connect-claude-code-with-tredict/)
- [ChatGPT](https://www.tredict.com/faq/connect-chatgpt-with-tredict/)
- [Codex](https://www.tredict.com/faq/connect-codex-with-tredict/)
- [Mistral Le Chat](https://www.tredict.com/faq/connect-mistral-le-chat-with-tredict/)
- [Mistral Vibe](https://www.tredict.com/faq/connect-mistral-vibe-with-tredict/)

## Configuration

### Claude.ai / Claude Desktop

Settings → Connectors → Add custom connector:

- Name: `Tredict`
- Remote MCP Server URL: `https://www.tredict.com/api/mcp/v2`

OAuth is handled automatically on first use.

### Generic MCP client (Streamable HTTP)

```json
{
  "mcpServers": {
    "tredict": {
      "url": "https://www.tredict.com/api/mcp/v2",
      "transport": { "type": "http" }
    }
  }
}
```

### Personal API Token

For clients without OAuth support, create an access token in Tredict at "Settings → Personal API / MCP" and pass it as a bearer token:

```
Authorization: Bearer <your-personal-api-token>
```

## Authentication

The server supports two methods:

1. **OAuth 2.1 with PKCE and Client Credentials.** Recommended for third-party MCP hosts. Scopes are fine-grained: `activityRead`, `activityWrite`, `bodyvaluesRead`.
2. **Personal API Token.** Recommended for self-built agents and local LLM workflows.

See the [OAuth2 documentation](https://www.tredict.com/blog/oauth_docs) for full details.

## Available tools

All tool names, descriptions and schemas can be listed by the MCP client. Scope annotations in parentheses.

### Activities

| Tool | Description | Scope |
|---|---|---|
| `activity-list` | List executed activities for a date range | activityRead |
| `activity` | Activity details, metrics and time series | activityRead |
| `activity-update` | Update title and notes of an activity (non-destructive) | activityWrite |
| `equipment-list` | List all user equipment (bikes, shoes, swimming, misc) | activityRead |

### Planning

| Tool | Description | Scope |
|---|---|---|
| `plan-creation` | Create a reusable training plan in "My Plans" | activityWrite |
| `add-plan-training` | Add a single structured workout to an existing plan | activityWrite |
| `planned-workout-list` | List planned / scheduled workouts | activityRead |
| `planned-workout` | Details and structure of a single planned workout | activityRead |
| `planned-workout-change-date` | Reschedule a planned workout to a new date | activityWrite |
| `training-effort-list` | Training effort over time across all sportTypes | activityRead |

### Body, health and capacity

| Tool | Description | Scope |
|---|---|---|
| `bodyvalues` | Weight, body fat, resting HR, body height and more | bodyvaluesRead |
| `capacity` | Capacity values (HRmax, HRlth, FTP, FTPace) per sportType | bodyvaluesRead |
| `hrv-list` | Heart rate variability data with baseline | bodyvaluesRead |
| `sleep-list` | Sleep data with baseline | bodyvaluesRead |
| `zones` | Zone revisions per sportType and zoneType | bodyvaluesRead |
| `zones-distribution` | Aggregated zones distribution per sportType and month | bodyvaluesRead |

### Interactive MCP App UI widgets

| Tool | Description | Scope |
|---|---|---|
| `show-activity-ui` | Render activity details, metrics, laps and map in the chat | activityRead |
| `show-plan-ui` | Render a reusable training plan with calendar view in the chat | activityRead |

## Prompts

Predefined, parameterised tasks the host can offer as templates:

- `better-names` : find recent activities with poor titles and suggest meaningful ones.
- `coolest-training` (year) : find the most interesting session of a given year.
- `compare-years` (firstYear, otherYear) : in-depth year-over-year comparison.
- `create-plan` (monthsBack?, durationInWeeks?, workoutsPerWeeks?, sportTypes?, targetZonesTypes?) : build a full training plan from recent history.
- `determine-ftp` (sportType?) : derive current FTP from training history.
- `indepth-analysis` : 2-year in-depth training status assessment.
- `recreate-structured-workout` (activityId) : rebuild an executable structured workout from an executed activity.

## Resources

| URI | Description |
|---|---|
| `mcp://tredict/assets/logo.svg` | Tredict rectangular logo (light / dark themes) |
| `mcp://tredict/assets/logo-slogan.svg` | Tredict logo with "Analytics for Endurance Athletes" slogan |
| `ui://widget/activity-ui-100.html` | Activity details UI widget (MCP App) |
| `ui://widget/plan-ui-100.html` | Training plan UI widget (MCP App) |

## Data integrity

- All write operations are reversible. `activity-update` stores previous titles and notes in the mcpUpdatedFields array, so any change can be rolled back at any time.
- The server only accesses your own profile. If you have access to other Tredict athletes as a friend or coach, their data remains invisible to the MCP server.

## Health data responsibility

Some responses, in particular from `plan-creation`, ask the LLM to surface health and liability notices. By registering with Tredict, the user accepts responsibility for health-related applications of the data as defined in the [Terms of Use](https://www.tredict.com/termsofuse/). LLMs make mistakes and are not a substitute for a qualified coach or your own judgement.

## Privacy and control

An external MCP client has no access to Tredict data until the user explicitly configures it. Read and write permissions can be restricted per scope at token creation. Tredict is fully GDPR compliant; see the [Privacy Policy](https://www.tredict.com/privacy/). Review the privacy policy of the connected AI provider for what happens to data sent through it, or use a local LLM to keep everything on-device.

## Links

- Server documentation: [MCP Server Docs](https://www.tredict.com/blog/mcp_server_docs/)
- Launch post with examples: [Use AI assistants and LLMs with the Tredict MCP Server](https://www.tredict.com/blog/use_ai_assistants_and_llms_with_the_tredict_mcp_server_and_interactive_mcp_apps/)
- FAQ: [Model Context Protocol and AI integration](https://www.tredict.com/faq/#model-context-protocol-and-ai-integration)
- Glossary: [Artificial Intelligence and MCP for endurance sports](https://www.tredict.com/glossary/artificial-intelligence-and-mcp/)