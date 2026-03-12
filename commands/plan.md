---
description: Spec-First research + implementation plan. Outputs SPEC (API/types/deps) AND step-by-step phases with dependency graph. WAIT for user CONFIRM before touching any code.
---

# Plan Command

This command invokes the **planner** agent to research the target technology, produce a **SPEC** (API design, data models, dependencies), and create a phased implementation plan — all before writing any code.

## What This Command Does

1. **Research** - Call `context7` + `exa-web-search` MCP to gather official docs and best practices
2. **Output SPEC** - API/interface design, data models, dependency table, compile verification plan
3. **Create Step Plan** - Break down into phases with `depends-on` relationships
4. **Wait for Confirmation** - MUST receive user approval before proceeding

> 💡 如果只需要 SPEC 不需要步骤分解，使用 `/spec` 快捷方式。

## When to Use

Use `/plan` when:
- Starting a new feature
- Making significant architectural changes
- Working on complex refactoring
- Multiple files/components will be affected
- Requirements are unclear or ambiguous

## How It Works

The planner agent will:

1. **Research** target technology via `context7` and `exa-web-search` MCP
2. **Output SPEC** with API design, data models, deps, compile plan, known limitations
3. **Break down into phases** with specific, actionable steps and `depends-on` graph
4. **Identify risks** and potential blockers
5. **Present the full plan** and WAIT for your explicit confirmation

## Example Usage

```
User: /plan I need to add real-time notifications when markets resolve

Agent (planner):
# Implementation Plan: Real-Time Market Resolution Notifications

## Requirements Restatement
- Send notifications to users when markets they're watching resolve
- Support multiple notification channels (in-app, email, webhook)
- Ensure notifications are delivered reliably
- Include market outcome and user's position result

## Implementation Phases

### Phase 1: Database Schema
- Add notifications table with columns: id, user_id, market_id, type, status, created_at
- Add user_notification_preferences table for channel preferences
- Create indexes on user_id and market_id for performance

### Phase 2: Notification Service
- Create notification service in lib/notifications.ts
- Implement notification queue using BullMQ/Redis
- Add retry logic for failed deliveries
- Create notification templates

### Phase 3: Integration Points
- Hook into market resolution logic (when status changes to "resolved")
- Query all users with positions in market
- Enqueue notifications for each user

### Phase 4: Frontend Components
- Create NotificationBell component in header
- Add NotificationList modal
- Implement real-time updates via Supabase subscriptions
- Add notification preferences page

## Dependencies
- Redis (for queue)
- Email service (SendGrid/Resend)
- Supabase real-time subscriptions

## Risks
- HIGH: Email deliverability (SPF/DKIM required)
- MEDIUM: Performance with 1000+ users per market
- MEDIUM: Notification spam if markets resolve frequently
- LOW: Real-time subscription overhead

## Estimated Complexity: MEDIUM
- Backend: 4-6 hours
- Frontend: 3-4 hours
- CDD Verification: 1-2 hours
- Total: 9-12 hours

**WAITING FOR CONFIRMATION**: Proceed with this plan? (yes/no/modify)
```

## Important Notes

**CRITICAL**: The planner agent will **NOT** write any code until you explicitly confirm the plan with "yes" or "proceed" or similar affirmative response.

If you want changes, respond with:
- "modify: [your changes]"
- "different approach: [alternative]"
- "skip phase 2 and do phase 3 first"

## Integration with Other Commands

After planning:
- Use `/build-fix` if build errors occur during implementation
- Use `/verify` to run CDD verification
- Use `/code-review` to review completed implementation
- Use `/spec` if you only need the SPEC without the full step plan

## Related Agents

This command invokes the `planner` agent provided by ECC.

For manual installs, the source file lives at:
`agents/planner.md`
