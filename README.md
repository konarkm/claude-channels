# claude-channels

Channel plugins for [Claude Code channels](https://code.claude.com/docs/en/channels) (research preview).

## photon — iMessage on a dedicated number

Two-way iMessage bridge for a Claude Code session via [Photon's Spectrum](https://photon.codes) cloud. Runs on a dedicated bot number — no Mac Messages.app, no Apple ID, no texting yourself. Photon's free tier covers personal use.

**Features**

- Text ↔ session bridge: messages land in your running session; Claude replies from the bot number
- Markdown rich text, file/image attachments both ways, native tapbacks, quote-reply threading, typing indicators, read receipts, iMessage send effects (confetti, slam, invisible ink, …)
- Group chats with per-group allowlists and mention-gating
- Permission relay: tool-approval prompts get texted to you; reply `yes abcde` / `no abcde`
- Remote session control by text: `/compact`, `/clear`, `/context` (context-window usage), `/peek` (snapshot of the terminal screen), `/restart` (exit + resume via `--continue`), `/restart fresh`
- Agent self-management tools: `check_context`, `compact_session`, `restart_session` (with `then_prompt` to continue work across the restart), and `queue_prompt` (queue its own next turn) — mutating ones go through the terminal input queue, so they take effect when the current turn ends, never mid-thought
- Access control: sender allowlist with a pairing-code flow (`/photon:access`), outbound restricted to allowlisted chats, channel state protected from exfiltration

**Requirements**

- Claude Code **v2.1.80+** with Anthropic auth (claude.ai or Console) — channels don't work on Bedrock/Vertex/Foundry; v2.1.187+ recommended (channel-connection fixes)
- [Bun](https://bun.sh) on your PATH — the channel server runs on it (`curl -fsSL https://bun.sh/install | bash`)
- macOS or Linux (the server uses `ps`/`lsof` for session detection)
- `tmux` — optional, but required for the remote `/compact`/`/clear`/`/restart`/`/peek` commands and the agent's `compact_session`/`restart_session`/`queue_prompt` tools

**Setup**

1. Create a project at [app.photon.codes](https://app.photon.codes), grab `projectId` + `projectSecret`.
2. Install:
   ```
   /plugin marketplace add konarkm/claude-channels
   /plugin install photon@claude-channels
   ```
3. Write `~/.claude/channels/photon/.env` (mode 600), or run `/photon:configure`:
   ```
   PHOTON_PROJECT_ID=<uuid>
   PHOTON_PROJECT_SECRET=<secret>
   ```
4. Allowlist the channel so it loads without the dev-flag consent prompt (you are the admin of your own machine — macOS path shown):
   ```bash
   sudo mkdir -p "/Library/Application Support/ClaudeCode" && sudo tee "/Library/Application Support/ClaudeCode/managed-settings.json" >/dev/null <<'EOF'
   {
     "channelsEnabled": true,
     "allowedChannelPlugins": [
       { "marketplace": "claude-channels", "plugin": "photon" }
     ]
   }
   EOF
   ```
   (Without this, use `claude --dangerously-load-development-channels plugin:photon@claude-channels` and accept the prompt each launch.)
5. Launch inside tmux so remote `/compact` / `/restart` can drive the terminal:
   ```bash
   tmux new -A -s claude
   claude --channels plugin:photon@claude-channels
   ```
   A dedicated alias keeps plain `claude` channel-free while making the channel session easy to launch: `alias claude-photon='claude --channels plugin:photon@claude-channels'`. The /restart relauncher retypes the full flags itself, so no alias is required for restarts.
6. Connect: `/photon:access allow +1<your number>`, then ask Claude in the session to text you (fresh Photon projects provision their line on first outbound send). Reply to the thread that arrives.

**Updating**

```
/plugin marketplace update claude-channels
/plugin uninstall photon@claude-channels
/plugin install photon@claude-channels
```

Then text `/restart` — the session relaunches on the new version by itself.

**Known limitations (Photon free-tier shared lines, as of mid-2026)**

- `edit_message` fails with a server error; `unsend_message` reports success without actually retracting
- Multi-part inbound attachments can arrive out of order
- The bot number comes from a shared pool and may differ across recipients; dedicated numbers are a paid Photon tier

**Security notes**

Anyone who can text the bot number can put words in front of your Claude session — the sender allowlist is the defense; don't loosen it. Permission relay only ever goes to allowlisted DMs. The server refuses to send its own state files (your project secret) as attachments, and access mutations must come from the terminal, never from channel messages.

## License

MIT
