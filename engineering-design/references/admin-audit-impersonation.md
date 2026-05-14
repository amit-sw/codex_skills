# Admin Audit and Impersonation Reference

Use this reference for privileged operations, admin tools, support actions, and impersonation.

## Impersonation Principles

Admin impersonation is allowed only when it is explicitly designed, role-gated, audited, and visible enough for later review.

Every impersonated action must record two identities:

- `admin_actor_id`: the real authenticated admin performing the action.
- `effective_user_id`: the user/account the admin is acting as.

Never overwrite one with the other.

## Required Impersonation Controls

- Cloudflare Access authenticates the admin.
- Application role checks verify the admin can impersonate.
- The admin must provide a reason or support ticket/case ID.
- The impersonation session has a short lifetime.
- UI should show that impersonation is active.
- Sensitive actions may require re-confirmation.
- Impersonation can be ended explicitly.
- Read-only impersonation should be preferred unless write access is necessary.

## Audit Event Fields

Store audit metadata compactly in D1. Store large before/after payloads in R2 if required.

Required fields:

- `id`
- `created_at`
- `request_id` or `trace_id`
- `event_type`
- `resource_type`
- `resource_id`
- `admin_actor_id` nullable for normal user actions
- `effective_user_id`
- `actor_role`
- `reason` or `ticket_id` for privileged support actions
- `outcome`
- `ip_hash` or coarse source metadata when useful
- `user_agent_hash` when useful
- `metadata_r2_key` for large details

Do not store secrets or full sensitive content in audit rows.

## Actions That Must Be Audited

- Login/auth failures when visible to the app.
- Role, permission, policy, or ownership changes.
- Admin impersonation start, stop, and every write while impersonating.
- Data export, import, deletion, restore, or bulk mutation.
- R2 object creation/deletion for user content.
- D1 schema or migration actions if performed through the app.
- LLM actions that make or support user-impacting decisions.
- Changes to prompts, agent tools, model settings, or provider credentials.

## Testing

Tests must prove:

- Non-admins cannot impersonate.
- Admins cannot impersonate without required reason/ticket.
- Expired impersonation sessions fail closed.
- Audit writes occur for success and failure paths.
- Both identities are preserved through route, service, repository, queue, and LLM workflows.
