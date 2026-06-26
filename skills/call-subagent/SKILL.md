---
name: call-subagent
description: |
  Interactive workflow to select an agent from ./agents/ (or ~/Projects/pi-subagnets/agents/),
  choose an LLM model for it, set a thinking level, and optionally invoke it.
  Use when the user says "call subagent", "run agent", "invoke agent", "set model for agent",
  "configure agent model", "launch agent", or asks to define/change which model an agent uses.
---

# Call Subagent

Interactive workflow for selecting, configuring, and invoking subagents with a chosen LLM model.

---

## Workflow

> **⚠️ Mandatory steps:** Steps 3 (Model Selection) and 4 (Thinking Level Selection) are **never optional**. You must always present the full selection menu for both, regardless of whether the user specified a model or thinking level in their request.

### Step 1 — Discover Agents

Find available agent `.md` files:

1. **Project agents** in `./agents/` (relative to cwd)
2. **User agents** in `~/Project/pi-subagents/agents/`

List them with their name, description, and current model (if any).

**Bash commands:**
```bash
ls -1 agents/*.md 2>/dev/null | while read f; do
  name=$(grep -m1 '^name:' "$f" | sed 's/^name:[[:space:]]*//')
  desc=$(grep -m1 '^description:' "$f" | sed 's/^description:[[:space:]]*//')
  model=$(grep -m1 '^model:' "$f" | sed 's/^model:[[:space:]]*//')
  echo "  $f → $name${model:+ (model: $model)} — $desc"
done
```

### Step 2 — Agent Selection

Ask the user which agent they want to configure/invoke. Present the list with index numbers.

### Step 3 — Model Selection (Always Required)

**You MUST always ask the user to choose a model — never skip this step, even if the user mentioned a model in their initial request.** If they did mention one, highlight it in the menu (e.g. mark it as "your suggestion") but still present all options and let them confirm or change their mind.

Present the menu of common options plus a custom entry:

| # | Model | Provider | Notes |
|---|-------|----------|-------|
| 1 | `deepseek/deepseek-v4-flash` | DeepSeek | Fast, cheap, good for research/coding |
| 2 | `deepseek/deepseek-v3.2` | DeepSeek | General-purpose |
| 3 | `moonshotai/kimi-v2.5` | Moonshot AI | General-purpose |
| 4 | `moonshotai/kimit-v2.7-code` | Moonshot AI | Strong at code/architecture |
| 5 | `z-ai/glm-5.2` | Z-AI | Good for code generation |
| 6 | `deepseek/deepseek-r1-0528` | DeepSeek | Strong reasoning, high thinking |
| 7 | `deepseek/deepseek-v4-pro` | DeepSeek | Most capable |
| 8 | **Custom** | — | User types any model ID |

If the user chooses Custom, prompt them to type the full model identifier (e.g. `provider/model-id`).

### Step 4 — Thinking Level Selection (Always Required)

**You MUST always ask the user to choose a thinking level — never skip this step.** Even if the user has a preferred level from a previous invocation or mentioned it in their request, always present the full menu and let them confirm.

| # | Level | Description |
|---|-------|-------------|
| 1 | `off` | No thinking output |
| 2 | `minimal` | Minimal reasoning |
| 3 | `low` | Low reasoning |
| 4 | `medium` | Balanced |
| 5 | `high` | Deep reasoning |
| 6 | `xhigh` | Maximum reasoning |

### Step 5 — Apply Configuration to Agent File

Edit the agent's `.md` file to add or update the `model` and `thinking` fields in the YAML frontmatter.

**Use the `edit` tool** with exact text matching. Two cases:

**Case A: `model` field already exists** — replace its value:
```
model: <old-value>
```
↓
```
model: <new-value>
```

**Case B: `model` field does not exist** — insert it after the `description` line (or after `name` if description is absent) and before `thinking` (or after an appropriate anchor). Find a unique anchor line to insert after. For example, if the file has:
```
description: Writes automated tests
```
Insert after it:
```
model: <new-value>
```

Same approach for the `thinking` field — update if exists, insert if not.

**Always read the file first** to verify current frontmatter before editing.

### Step 6 — Confirm and Optionally Invoke

Show the user a summary of what was configured:

```
Agent: <name>
 File: <path>
 Model: <model>
Thinking: <thinking>
```

Ask if they want to invoke the agent now:
- **Yes** — use the `subagent` tool: `{ agent: "<name>", task: "<user's original task or ask for a task>" }`
- **No** — just confirm the changes were saved.

---

## Examples

**User:** "call subagent test-designer with claude sonnet"
**You:** Step through the workflow. Even though they mentioned `claude sonnet`, still present the full model menu with `anthropic/claude-sonnet-4-20250514` highlighted as their suggestion, let them confirm or pick another model, then always present the thinking level menu for them to choose.

**User:** "run tester on gemini flash"
**You:** Select the `tester` agent, still present the full model menu with `google/gemini-2.5-flash` highlighted, then always ask for thinking level, then offer to invoke.

**User:** "set model for my agent"
**You:** List all discovered agents, let them pick one, then always proceed with model selection and thinking level selection.
