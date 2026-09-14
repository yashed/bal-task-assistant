# Task Assistant Agent (Ballerina) - Deployment Guide

## Overview

A to-do list assistant built with [Ballerina](https://ballerina.io)'s `ai` module and OpenAI. The agent manages an in-memory task list through six tools — add, list, complete, reschedule, delete, and read the current date — and is instructed to resolve relative dates ("tomorrow", "next Friday") against the current date before acting on them. The service exposes `POST /chat` on port `8000`.

This is the reference sample for Agent Manager's **Ballerina buildpack**. Unlike the other samples in this repository, it's built directly from source with no Dockerfile, and — because Ballerina buildpacks don't need one — no language version or start command to configure.

## Prerequisites

### Required API Keys

- **OpenAI API Key**: for model inference (the agent uses `gpt-4o`)

## Deployment Instructions

### Step 1: Access Agent Manager

1. Navigate to the **Default** project
2. Click **"Add Agent"**
3. Select the **Platform-Hosted Agent** card
4. Pick **Source Code** as the source type

### Step 2: Configure Agent Details

Fill in the agent creation form with these values:

| Field | Value |
| --- | --- |
| **Display Name** | `Task Assistant` |
| **Description** | `Ballerina to-do list assistant` |
| **GitHub Repository** | `https://github.com/wso2/agent-manager` |
| **Branch** | `main` |
| **App Path** | `samples/bal-task-assistant` |
| **Language** | `Ballerina` |
| **Port** | `8000` |

Ballerina is the one buildpack that doesn't ask for a **Language Version** or **Start Command** — the platform builds and runs `main.bal` directly, and the fields don't appear once you select it.

### Step 3: Select Agent Interface

- Choose **"Chat Agent"** as the agent interface type

### Step 4: Configure Environment Variables

Ballerina reads `configurable` values from environment variables named `BAL_CONFIG_VAR_<NAME>`, where `<NAME>` is the configurable's identifier, uppercased. This agent declares one configurable, `openAiApiKey`, so set:

```env
BAL_CONFIG_VAR_OPENAIAPIKEY=<your-openai-api-key>
```

This is different from every Python sample in this repository (which read a plain `OPENAI_API_KEY`) — it's the same convention the [AMP instrumentation guide](../../documentation/docs/guides/amp-instrumentation.mdx) uses for `ballerinax/amp`'s own configurables, just applied to this agent's own key instead.

### Step 5: Deploy the Agent

1. Review all configuration details
2. Click **"Deploy"**
3. Wait for the build to complete

## Testing Your Agent

### Step 1: Navigate to Chat Interface

Click on the **"Try It"** section on the left navigation.

### Step 2: Test Sample Interactions

Try these in order — they exercise every tool, including the two that read a task back by the id a previous reply gave you:

```text
List all my tasks
```

```text
Add a task to call the dentist tomorrow
```

```text
Mark the call the dentist task as done
```

```text
Push the groceries task to next Monday
```

```text
Remove the groceries task, I don't need it anymore
```

### Step 3: Observe Traces

1. Click on the **"Observability"** tab on the left navigation and select **Traces**
2. Open a trace to see the tool calls chosen for each message — `getCurrentDate` should appear before any tool that took a due date, since the system prompt asks the agent to resolve relative dates first

## Run Locally

```bash
cd samples/bal-task-assistant
echo 'openAiApiKey = "<your-openai-api-key>"' > Config.toml
bal run    # serves on http://localhost:8000
```

```bash
curl -s localhost:8000/chat \
  -H 'content-type: application/json' \
  -d '{"sessionId": "s1", "message": "Add a task to call the dentist tomorrow"}'
```

`Config.toml` is git-ignored — never commit real credentials to it.

## Notes

- Task ids are returned by `addTask` and `listTasks`. The system prompt tells the agent to reuse an id from an earlier reply rather than ask the user for one, which is why the test sequence above refers to tasks by description and still works — the agent resolves the description to an id itself via `listTasks`.
- The in-memory task store resets on every restart — there's no database. That's deliberate: the point of this sample is the Ballerina buildpack and the `ai:Agent` / `@ai:AgentTool` pattern, not persistence.
- `ballerinax/amp` is imported for its side effect only (`import ballerinax/amp as _;`) — it wires up AMP's OpenTelemetry auto-instrumentation with no code changes beyond the import.
