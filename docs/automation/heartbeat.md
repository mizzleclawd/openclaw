# Heartbeat System for OpenClaw

A cron-based system for autonomous agent coordination.

---

## Overview

The Heartbeat System allows OpenClaw agents to:
- Wake up on a schedule
- Check for work (tasks, mentions, messages)
- Perform work or report HEARTBEAT_OK
- Sleep until next cycle

This enables 24/7 autonomous operation without burning API credits when idle.

---

## HEARTBEAT.md Template

Create a `HEARTBEAT.md` file in your workspace:

```markdown
# Heartbeat Tasks - Periodic Checks

## On Wake
- [ ] Read memory/WORKING.md for ongoing tasks
- [ ] If task in progress, resume it
- [ ] Search session memory if context unclear

## Periodic Checks (every 15 minutes)
- [ ] Check shared-brain for @mentions
- [ ] Check assigned tasks
- [ ] Scan activity feed for relevant discussions

## Response Rules
- If work needed → Do it
- If nothing → Reply HEARTBEAT_OK
```

---

## Cron Setup

Add heartbeat crons to OpenClaw:

```bash
# Agent wakes every 15 minutes
openclaw cron add \
  --name "heartbeat-check" \
  --cron "*/15 * * * *" \
  --session "isolated" \
  --message "Check HEARTBEAT.md and report status."
```

### Staggered Schedule Example

```bash
# Agent 1: wakes at :00, :15, :30, :45
# Agent 2: wakes at :02, :17, :32, :47
# Agent 3: wakes at :04, :19, :34, :49
```

This prevents all agents from waking simultaneously.

---

## What Happens During a Heartbeat

1. **Load Context**
   - Read WORKING.md
   - Read recent daily notes
   - Check session memory

2. **Check for Urgent Items**
   - @mentions
   - Assigned tasks
   - Critical notifications

3. **Scan Activity Feed**
   - Relevant discussions
   - Updates to watched items
   - Blockers or escalations

4. **Take Action or Stand Down**
   - If work → execute
   - If nothing → HEARTBEAT_OK

---

## Coordination Patterns

### Shared Brain

Multiple agents share context via a common data store:

```
shared-brain/
├── handoffs/      # Agent-to-agent communication
├── projects/      # Active projects
├── tasks/         # Task tracking
└── agents/        # Agent SOUL files
```

### @Mentions

Type @AgentName to notify a specific agent:

```markdown
@Vision Can you review this SEO strategy?
```

The agent receives the notification on their next heartbeat.

### Task Routing

```markdown
Request comes in
       ↓
Route to right agent
       ↓
Agent wakes, checks context
       ↓
Agent does work
       ↓
Report status
```

---

## Daily Standup Example

Agents can generate daily summaries:

```markdown
📊 DAILY STANDUP — Feb 5, 2026

✅ COMPLETED
- Agent 1: Task A completed
- Agent 2: Task B completed

🔄 IN PROGRESS
- Agent 1: Task C in progress
- Agent 2: Waiting on blocker

🚫 BLOCKED
- Agent 1: Needs input from Darius
```

---

## Benefits

| Benefit | Description |
|---------|-------------|
| **Cost Efficiency** | Agents only burn credits when working |
| **24/7 Coverage** | Autonomous operation without manual intervention |
| **Coordination** | Multiple agents work together seamlessly |
| **Visibility** | Clear status on what's happening |

---

## Anti-Patterns to Avoid

| Anti-Pattern | Solution |
|--------------|----------|
| All agents wake at once | Stagger cron schedules |
| No shared context | Use shared-brain or database |
| No clear tasks | Define tasks explicitly |
| Missing notifications | Implement @mention system |

---

## Example Configuration

```json
{
  "agents": {
    "jarvis": {
      "schedule": "*/15 * * * *",
      "session": "agent:main:main",
      "heartbeat": true
    },
    "shuri": {
      "schedule": "2-59/15 * * * *",
      "session": "agent:product-analyst:main",
      "heartbeat": true
    }
  }
}
```

---

*Inspired by Mission Control architecture (pbteja1998)*
