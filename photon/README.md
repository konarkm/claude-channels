# Photon iMessage channel for Claude Code

Two-way iMessage bridge for a Claude Code session, running on a **dedicated
Photon-provisioned iMessage number** via the [Spectrum](https://photon.codes)
cloud — no tunnel, no Messages.app, no Apple ID involvement, and no
"texting yourself" problem.

## What it does

- Text the bot number from your phone; the message lands in your running
  Claude Code session as a `<channel>` event.
- Claude replies through the same number: plain text, markdown rich text,
  file attachments, tapback reactions, quote-reply threading, message edits
  (~15 min window), unsend (~2 min window), typing indicators, read
  receipts, and iMessage send effects (confetti, slam, invisible ink, …).
- Inbound images and attachments are downloaded to
  `~/.claude/channels/photon/inbox/` and handed to Claude as file paths.
  Voice notes are saved as audio files.
- Group chats are supported with per-group allowlists and
  mention-triggering ("claude ..." or quote-replying the bot).
- Permission relay: when Claude Code opens a tool-approval dialog, the
  prompt is also texted to every paired handle. Reply `yes abcde` /
  `no abcde` from your phone to approve or deny remotely.
- Remote session control: text `/compact`, `/clear`, or `/restart` and the
  server drives the terminal directly via `tmux send-keys` (never reaches
  Claude). Requires the session to run inside tmux:
  `tmux new -A -s claude`, then `claude`. `/restart` exits and relaunches
  via your shell (an alias must re-add the channels flag) and texts you
  when the session is back online.

## Setup

1. Create a project at [app.photon.codes](https://app.photon.codes) and grab
   `projectId` + `projectSecret` from Settings.
2. Write `~/.claude/channels/photon/.env` (or run `/photon:configure`):

   ```
   PHOTON_PROJECT_ID=<uuid>
   PHOTON_PROJECT_SECRET=<secret>
   ```

   `chmod 600` it.
3. Install the plugin and start a session with the channel enabled:

   ```bash
   claude --dangerously-load-development-channels plugin:photon@claude-channels
   ```

   (Channels are a research preview; custom channels need the development
   flag until they're on an allowlist.)
4. Get connected. Two paths:

   - **Outbound-first (fresh projects):** Photon auto-provisions the line on
     first send (`autoScale`), so there may be no number to text yet. Run
     `/photon:access allow +1<your number>`, then ask Claude in the session
     to send you a text (reply tool, `chat_id` = your number). Reply to the
     thread that arrives — you're connected.
   - **Inbound-first:** if your Photon dashboard already shows a number for
     the project, text it. You'll get a pairing code back; run
     `/photon:access pair <code>`.

## Access control

Inbound messages are gated on the **sender's handle** against an allowlist in
`~/.claude/channels/photon/access.json`, managed by `/photon:access`
(pairing approvals, direct allows, DM policy, group registration, delivery
options). Unknown senders get at most two pairing prompts, then silence.
Without the gate, anyone who discovers the bot number could inject prompts
into your session — don't loosen it.

## State

Everything lives in `~/.claude/channels/photon/`:

| File | Purpose |
| --- | --- |
| `.env` | Photon credentials (mode 600) |
| `access.json` | allowlist, policies, delivery options |
| `inbox/` | downloaded inbound attachments |
| `approved/` | pairing-confirmation handshake with `/photon:access` |
