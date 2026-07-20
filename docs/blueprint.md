# TripSplit — Bot specification

**Archetype:** finance

**Voice:** professional and concise — write every user-facing message, button label, error, and empty state in this voice.

A Telegram group bot that tracks shared trip expenses, splits costs among participants, and maintains mathematically consistent balances with confirmation-based settlements. Supports late joiners, immutable expense records, and organizer-controlled participant management.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- friends
- families
- small travel groups

## Success criteria

- Accurate balance calculations with deterministic rounding
- Immutable expense records with audit trail
- Confirmation-based payment settlements
- Organizer-controlled participant management
- Minimal transaction settlement suggestions

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Open main menu with trip overview and balance commands
- **/newtrip** (command, actor: user, command: /newtrip) — Create a new trip with optional currency specification
- **/jointrip** (command, actor: user, command: /jointrip) — Join an active trip as participant
- **/leavetrip** (command, actor: user, command: /leavetrip) — Leave current trip
- **/expense** (command, actor: user, command: /expense) — Record a new expense with payer, amount, participants, and split type
- **/balance** (command, actor: user, command: /balance) — View current balance summary and suggested settlements
- **/paid** (command, actor: user, command: /paid) — Mark a payment as completed with counterparty confirmation
- **/close** (command, actor: user, command: /close) — Close trip (organizer only)

## Flows

### Trip creation
_Trigger:_ /newtrip

1. Organizer creates trip with name and currency
2. Bot lists current group members as participants
3. Organizer can add/remove participants later

_Data touched:_ Trip, Participant

### Expense recording
_Trigger:_ /expense

1. User specifies amount, payer, participants, and split type
2. Bot applies deterministic rounding and records immutable expense
3. Bot posts group confirmation with balance summary

_Data touched:_ Expense, Balance snapshot

### Settlement confirmation
_Trigger:_ /paid

1. User records payment completion
2. Bot sends DM confirmation to counterparty
3. Settlement marked confirmed only after mutual confirmation

_Data touched:_ Payment

### Balance viewing
_Trigger:_ /balance

1. User requests balance summary
2. Bot displays net positions and minimal settlement suggestions
3. Organizer gets detailed expense list on demand

_Data touched:_ Balance snapshot, Expense

### Trip closure
_Trigger:_ /close

1. Organizer marks trip as closed
2. Trip becomes read-only but remains visible
3. Participants can still view balances

_Data touched:_ Trip

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **Trip** _(retention: persistent)_ — Trip metadata and participant list
  - fields: name, currency, organizer, participant_list, creation_timestamp, status
- **Participant** _(retention: persistent)_ — User in trip with join/leave timestamps
  - fields: telegram_user_id, display_name, join_timestamp, leave_timestamp, metadata
- **Expense** _(retention: persistent)_ — Immutable expense record with split details
  - fields: id, payer, amount, currency, description, timestamp, participants, share_weights, rounding_adjustments, recorded_by
- **Payment** _(retention: persistent)_ — Settlement confirmation with status tracking
  - fields: from, to, amount, timestamp, reference, status
- **Balance snapshot** _(retention: session)_ — Computed ledger of net positions
  - fields: participant, net_owes, simplified_settlements

## Integrations

- **Telegram** (required) — Bot API messaging
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- Create/delete trips
- Add/remove participants
- View detailed expense reports
- Create administrative adjustments

## Notifications

- Group chat expense confirmations
- Direct message payment confirmations
- Trip closure notifications

## Permissions & privacy

- Trip details visible only to participants
- Expense details visible only to participants
- DM confirmations for sensitive actions

## Edge cases

- Mid-trip joiners excluded from prior expenses
- Deterministic rounding for unequal splits
- Payment confirmation requires counterparty approval
- Immutable expense records with adjustment entries

## Required tests

- End-to-end expense recording with rounding
- Settlement confirmation workflow
- Balance calculation consistency
- Trip closure permissions
- Late joiner visibility rules

## Assumptions

- Default currency from group locale
- Rounding distributed by ascending user ID
- Organizer has final control over participants
- No external payment integrations
