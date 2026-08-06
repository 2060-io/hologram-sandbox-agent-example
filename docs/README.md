# Hologram Example Agent — User Guide

Welcome to the **Hologram Example Agent**, an AI assistant deployed as a Verifiable Service on the [Hologram](https://hologram.zone) + [Verana](https://verana.io) ecosystem. It's the canonical demo used by the [Hologram docs](https://docs.hologram.zone) and a starting point for developers building their own verifiable AI agents.

## Getting started

### 1. Install Hologram Messaging

Download the Hologram Messaging app from the [App Store](https://apps.apple.com/us/app/hologram-messaging/id6474701855) or [Google Play](https://play.google.com/store/apps/details?id=io.twentysixty.mobileagent.m) and create your account.

### 2. Get your Avatar credential

The agent uses credential-based authentication — you need a **Hologram Sandbox Avatar** credential to chat with it.

1. Open Hologram Messaging on your phone.
2. Navigate to `https://avatar.sandbox.hologram.zone/`.
3. Scan the QR code, choose a display name, and accept the credential offer.

### 3. Connect to the agent

Scan the QR code at `https://example-agent.sandbox.hologram.zone/`. The agent will greet you and ask you to authenticate.

### 4. Authenticate

Open the contextual menu (hamburger icon) and tap **Authenticate**. The agent will request your Avatar credential — accept the proof request and you're in.

## What can the agent do?

The agent has **Context7 MCP** plugged in, which gives it access to up-to-date documentation for thousands of libraries and frameworks. Once you're authenticated, you can ask things like:

- **Hologram / Verana questions** — "What is a Verifiable Public Registry?", "How does Hologram authenticate users?"
- **Library docs lookup** — "Show me the Next.js routing docs", "How do I use `useEffect` in React?", "What's the latest way to configure Tailwind?"
- **Framework comparisons** — "What's the difference between Prisma and Drizzle?"
- **Quick code patterns** — "How do I set up NestJS with TypeORM and PostgreSQL?"

Just type in natural language. The agent figures out which Context7 tool to call.

## Tips

- **Multi-language.** English and Spanish are supported out of the box. The agent responds in your language.
- **Fresh data.** Context7 fetches live documentation, so answers reflect the current state of each library.
- **Fork me.** This agent is an example. The source is at [`2060-io/hologram-ai-agent-example`](https://github.com/2060-io/hologram-ai-agent-example) — fork it to build your own agent with a different MCP, persona, or LLM.

## Troubleshooting

| Problem | Solution |
|---|---|
| "Authentication required" | Open the menu and tap **Authenticate**, then accept the credential request |
| The agent never gets my credential | Make sure you obtained the Avatar credential at `avatar.sandbox.hologram.zone` first |
| The agent is unresponsive | Try sending "hello" or disconnect and reconnect from Hologram Messaging |
| "Tool failed" or empty results | The Context7 service may be momentarily unavailable — retry in a few seconds |

## Privacy & security

- All communication is end-to-end encrypted via DIDComm.
- The agent only learns your Avatar credential's `name` (which you chose) — no phone numbers, no emails.
- Each session is isolated; no data is shared with other users.
- The agent's identity is cryptographically verifiable through the Verana testnet trust registry.
