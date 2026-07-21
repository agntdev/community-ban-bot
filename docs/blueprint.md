# Community Ban Bot — Bot specification

**Archetype:** community

**Voice:** professional and concise — write every user-facing message, button label, error, and empty state in this voice.

A Telegram bot that lets admins of a community group ban members manually via simple commands.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- Community group admins

## Success criteria

- Admins can ban disruptive members with /ban <member> [reason]
- Admins can unban members with /unban <member>
- Admins can list current bans with /bans
- Bans persist in SQLite across bot restarts
- Only admins can use ban commands
- Banned members are removed silently without notifications

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/ban** (command, actor: admin, command: /ban) — Ban a member with optional reason
  - inputs: member (user mention or ID), reason (optional)
  - outputs: Confirmation message, Ban record stored in SQLite
- **/unban** (command, actor: admin, command: /unban) — Restore a banned member
  - inputs: member (user mention or ID)
  - outputs: Confirmation message, Ban record removed from SQLite
- **/bans** (command, actor: admin, command: /bans) — List all current bans
  - outputs: List of banned members with ban timestamps

## Flows

### Ban flow
_Trigger:_ /ban <member> [reason]

1. Admin sends /ban command with member and optional reason
2. Bot checks admin status in the group
3. Bot checks if member is already banned
4. Bot removes member from the group
5. Bot stores ban record in SQLite with timestamp and reason
6. Bot sends confirmation message to admin

_Data touched:_ Ban

### Unban flow
_Trigger:_ /unban <member>

1. Admin sends /unban command with member
2. Bot checks admin status in the group
3. Bot checks if member is currently banned
4. Bot restores member to the group
5. Bot removes ban record from SQLite
6. Bot sends confirmation message to admin

_Data touched:_ Ban

### List bans flow
_Trigger:_ /bans

1. Admin sends /bans command
2. Bot checks admin status in the group
3. Bot retrieves all ban records from SQLite
4. Bot formats and sends list of banned members to admin

_Data touched:_ Ban

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **Ban** _(retention: persistent)_ — Record of an admin-initiated member removal
  - fields: member_id, member_username, ban_timestamp, reason, admin_id, admin_username

## Integrations

- **Telegram** (required) — Bot API messaging and group management
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- Add/remove admin permissions for the bot
- View ban history via /bans
- Manually unban members via /unban
- Configure ban reason defaults
- Review ban records in SQLite database

## Permissions & privacy

- Bot requires admin permissions in the community group to ban/unban members
- Bot stores member IDs and usernames in SQLite for ban records
- Bot does not store personal data beyond ban records
- Bot does not share ban data with external services

## Edge cases

- Member is already banned
- Member is not in the group
- Member is an admin (cannot be banned)
- Member is the bot itself
- Member leaves the group before ban is executed
- Admin is not an admin in the group
- SQLite database is corrupted or missing
- Member username changes between ban and unban

## Required tests

- Admin can ban a member with /ban <member> [reason]
- Admin can unban a member with /unban <member>
- Admin can list bans with /bans
- Non-admin users cannot use ban commands
- Banned members are removed from the group
- Ban records persist in SQLite
- Bans are permanent until manually unbanned
- No notifications sent to banned members or group
- Bot checks admin status before executing commands

## Assumptions

- Only admins with group admin rights can use ban commands
- Bot operates in one community group per instance
- Bans are permanent until manually unbanned
- Banned members are removed silently without notifications
- SQLite storage is used for ban records
- No automatic spam detection or moderation
- No user self-service for unbanning
