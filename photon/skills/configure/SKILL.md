---
name: configure
description: Configure the Photon iMessage channel — set Photon project credentials and verify the connection. Use when the user asks to set up, configure, or fix credentials for the Photon/iMessage channel.
user-invocable: true
allowed-tools:
  - Read
  - Write
  - Bash(ls *)
  - Bash(mkdir *)
  - Bash(chmod *)
---

# /photon:configure — Photon Channel Setup

Configures credentials for the Photon (Spectrum) iMessage channel. The channel
server reads them from `~/.claude/channels/photon/.env`.

Arguments passed: `$ARGUMENTS`

---

## Steps

1. **Get credentials.** The user needs a Photon project from
   https://app.photon.codes → project Settings: a `projectId` (UUID) and a
   `projectSecret`. If `$ARGUMENTS` contains them (two tokens: id then secret,
   or `PHOTON_PROJECT_ID=... PHOTON_PROJECT_SECRET=...` pairs), use those.
   Otherwise ask the user to paste them.

2. **Write the env file.** Create `~/.claude/channels/photon/` if missing
   (mode 700), then write `~/.claude/channels/photon/.env`:

   ```
   PHOTON_PROJECT_ID=<uuid>
   PHOTON_PROJECT_SECRET=<secret>
   ```

   Then `chmod 600 ~/.claude/channels/photon/.env`.

3. **Explain next steps** to the user:
   - Restart Claude Code with the channel enabled:
     `claude --dangerously-load-development-channels plugin:photon@claude-channels`
   - Text the project's Photon iMessage number from their phone. The first
     message gets a pairing code back.
   - Run `/photon:access pair <code>` to approve themselves.

## Security

Never print the secret back into the conversation transcript beyond
confirming it was written. If the `.env` already exists, show only the
project ID when confirming an overwrite.
