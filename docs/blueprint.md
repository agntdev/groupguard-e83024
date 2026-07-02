# GroupGuard — Bot specification

**Archetype:** community

**Voice:** professional and concise — write every user-facing message, button label, error, and empty state in this voice.

GroupGuard is a Telegram moderation bot that automates community management by verifying new members with a one-click human confirmation, detecting spam (links from new accounts, repeated messages, flooding), and enforcing admin-configurable thresholds (warn, mute, kick, ban). It maintains a transparent action log, supports trusted user exceptions, and provides periodic summaries of moderation activity while respecting admin and pinned message protections.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- Telegram group owners
- Community moderators

## Success criteria

- Reduces spam messages in group chats
- Enforces configured moderation thresholds automatically
- Maintains auditable action logs for 90 days by default
- Provides admin-configurable greeting and rules text

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Open main menu with moderation status and configuration options
- **Я человек** (button, actor: user, callback: verify:human) — Confirm human membership after joining
  - inputs: Telegram user ID
  - outputs: Verification status update, Restriction removal
- **/warn** (command, actor: admin, command: /warn) — Issue warning to user with optional reason
- **/mute** (command, actor: admin, command: /mute) — Mute user for specified duration
- **/kick** (command, actor: admin, command: /kick) — Remove user from group with reason
- **/ban** (command, actor: admin, command: /ban) — Permanently ban user with reason
- **/trust** (command, actor: admin, command: /trust) — Mark user as trusted (exempt from automod)
- **/untrust** (command, actor: admin, command: /untrust) — Remove user's trusted status
- **/set_greeting** (command, actor: admin, command: /set_greeting) — Update greeting and rules text
- **/set_threshold** (command, actor: admin, command: /set_threshold) — Configure moderation thresholds with guided defaults
- **/log** (command, actor: admin, command: /log) — View action log summary for specified period

## Flows

### new_member_verification
_Trigger:_ user_join

1. Send greeting with verification button
2. Wait for button click or timeout
3. Apply restriction removal or kick

_Data touched:_ User, New join session

### spam_detection
_Trigger:_ message_post

1. Check link age threshold
2. Count message repetitions
3. Monitor message rate
4. Apply configured thresholds
5. Post explanation and log action

_Data touched:_ User, Rule/threshold set, Action log entry

### admin_command_handling
_Trigger:_ /warn /mute /kick /ban /trust /untrust /set_greeting /set_threshold /log

1. Verify admin status
2. Validate command parameters
3. Execute action
4. Update log and chat

_Data touched:_ User, Rule/threshold set, Action log entry

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **User** _(retention: persistent)_ — Telegram group member with moderation status
  - fields: Telegram ID, Display name, Join timestamp, Trust flag, Role
- **New join session** _(retention: session)_ — Pending verification state for new members
  - fields: Telegram ID, Expiry timestamp, Restriction status
- **Rule/threshold set** _(retention: persistent)_ — Moderation policy configuration
  - fields: Link age threshold, Repeat count threshold, Flood rate threshold, Verification timeout, Action sequence
- **Action log entry** _(retention: persistent)_ — Record of moderation actions
  - fields: Actor (auto/admin), Target user, Reason, Action taken, Timestamp

## Integrations

- **Telegram** (required) — Bot API messaging
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- /set_threshold
- /set_greeting
- /trust
- /untrust
- /log

## Notifications

- Visible action explanations in group chat
- Private admin log accessible via /log
- Periodic summaries of joins/verifications/removals

## Permissions & privacy

- Admin commands require verified admin status
- No personal data stored beyond Telegram ID and display name
- Action logs only visible to admins via /log command
- All moderation actions comply with Telegram's API policies

## Edge cases

- Admins/trusted users never restricted
- Commands targeting other admins rejected with explanation
- Pinned messages never modified
- Expired verification sessions cleaned up automatically

## Required tests

- Verify new member verification flow with timeout handling
- Test spam detection thresholds and action sequences
- Validate admin command permissions and error handling
- Confirm action log persistence and queryability

## Assumptions

- Verification button is only 'Я человек' as specified
- Default thresholds follow progressive discipline model
- Log retention period is 90 days by default
