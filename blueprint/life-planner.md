# Life Planner

## Overview
The Life Planner is a unified task and goal scheduling tool. Users can create plans across multiple life contexts — **Work**, **Hobby**, **Life Goals**, and any **custom tags** they define — and view them all in one place or filter down to a specific context.

---

## Access by Tier

| Tier | Access |
|---|---|
| Public | None |
| Free | None |
| Investor | **Full access** within their own Server — real, persistent plans |

The Life Planner is a Server Feature. It is enabled per-Server by the Server owner via their Feature subscription.

---

## Core Concepts

### Plan (Task / Goal)
The primary unit. Represents a scheduled task, a goal, or a recurring commitment.

| Field | Type | Description |
|---|---|---|
| `id` | uuid | |
| `user_id` | FK → users | Owner |
| `server_id` | FK → servers | |
| `title` | string | |
| `description` | text | nullable, rich text |
| `context` | enum | `work` \| `hobby` \| `life_goal` \| `custom` |
| `tag_id` | FK → plan_tags (nullable) | Custom tag when context = `custom` |
| `status` | enum | `pending` \| `in_progress` \| `completed` \| `cancelled` |
| `priority` | enum | `low` \| `medium` \| `high` |
| `due_at` | timestamp | nullable |
| `starts_at` | timestamp | nullable (for scheduled blocks) |
| `is_recurring` | boolean | |
| `recurrence_rule` | json | nullable, same shape as Expense Planner |
| `completed_at` | timestamp | nullable |
| `notes` | text | nullable |
| `created_at` / `updated_at` | timestamps | |

### Plan Tags
Reusable custom tags that supplement the built-in contexts.

| Field | Type | Description |
|---|---|---|
| `id` | uuid | |
| `user_id` | FK → users | |
| `name` | string | e.g. "Side Project", "Reading", "Fitness" |
| `colour` | string | Hex |
| `icon` | string | nullable |

### Built-in Contexts
| Context | Description |
|---|---|
| `work` | Professional tasks, projects, deadlines |
| `hobby` | Personal interests, creative projects |
| `life_goal` | Long-term aspirations, milestones |
| `custom` | Anything outside the above; requires a tag |

---

## Features

### Unified View (All Plans)
- Shows all plans across all contexts in a single timeline or list.
- Grouped or colour-coded by context/tag.
- Filters: context, tag, status, priority, date range.
- Sort: due date, priority, created date.

### Context / Tag Views
- Dedicated filtered view per context (Work, Hobby, Life Goals).
- Custom-tag views per user-defined tag.
- Each view shows only plans belonging to that context/tag.

### Plan Detail
- Full detail page: title, description, status, priority, due date, notes.
- Checklist / sub-tasks within a plan (see Sub-tasks below).
- Activity log: status changes, edits.

### Sub-tasks
- A plan can have ordered sub-tasks (simple checklist items).
- Sub-task completion contributes to a progress indicator on the parent plan.

```
plan_subtasks
  id          uuid
  plan_id     FK → plans
  title       string
  is_done     boolean
  order       integer
```

### Calendar View
- Monthly/weekly calendar showing all plans with due/start dates.
- Toggle contexts on/off for a personalised view.
- Colour-coded by context (and tag for custom).

### Timeline / Gantt View (Server Feature)
- Horizontal timeline view showing plans with start and due dates as bars.
- Useful for Life Goal tracking and project planning.
- Gated behind the Server Feature subscription.

### Recurring Plans
- Same recurrence mechanism as Expense Planner (shared `recurrence_rule` JSON shape).
- e.g. "Weekly team standup", "Monthly review", "Annual goal check-in".
- Occurrences are generated ahead of time; each occurrence can be independently completed.

### Team-Shared Plans (Server Feature)
- Investor can create plans visible/shared within their Server or a specific Group.
- Group members can be assigned to shared plans.
- Shared plans show an "assignees" field.
- Gated behind the Server Feature subscription.

---

## Permissions

| Permission | `super_owner` | `investor` | `free` |
|---|---|---|---|
| `plans.read` | ✅ | ✅ (own Server) | ❌ |
| `plans.create` | ✅ | ✅ (own Server) | ❌ |
| `plans.update` | ✅ | ✅ (own Server) | ❌ |
| `plans.delete` | ✅ | ✅ (own Server) | ❌ |
| `plans.tags.manage` | ✅ | ✅ (own Server) | ❌ |
| `plans.shared` | ✅ | ✅ (Server Feature) | ❌ |
| `plans.timeline_view` | ✅ | ✅ (Server Feature) | ❌ |

---

## Views Summary

| View | Available To |
|---|---|
| Unified list / all contexts | Investor, Super Owner |
| Per-context filter view | Investor, Super Owner |
| Custom tag view | Investor, Super Owner |
| Calendar | Supporter, Investor |
| Timeline / Gantt | Investor (Ability) |
| Team-shared plans | Investor (Ability) |

---

## Open Questions
- Should sub-tasks support their own due dates, or just be a checklist?
- Is drag-and-drop reordering needed for the plan list / calendar?
- Should Life Goals support progress percentage (manual or derived from sub-tasks)?
- Notifications / reminders for upcoming or overdue plans — email, in-app, or both?
- Should Free users see a demo/preview of the planner, or just an upsell screen?
- Are work plans ever collaborative for Supporter orgs, or only for Investors?
