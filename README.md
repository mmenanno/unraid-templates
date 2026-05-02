# Unraid Templates

[![Lint](https://github.com/mmenanno/unraid-templates/actions/workflows/lint.yml/badge.svg)](https://github.com/mmenanno/unraid-templates/actions/workflows/lint.yml)
[![License: MIT](https://img.shields.io/github/license/mmenanno/unraid-templates)](./LICENSE)

Unraid Community Applications templates for containers I publish.

---

## RedditChatBridge

<img src="https://raw.githubusercontent.com/mmenanno/reddit_chat_bridge/main/app/assets/icons/icon-1024.png" width="96" align="right" alt="RedditChatBridge icon">

Self-hosted bridge between Reddit Chat and a private Discord server. Reddit DMs land in per-conversation `#dm-*` channels with the sender's snoovatar; replies relay back under your Reddit identity. All config lives in a bcrypt-protected admin web UI.

- **Project:** [mmenanno/reddit_chat_bridge](https://github.com/mmenanno/reddit_chat_bridge)
- **Template:** [`templates/reddit_chat_bridge.xml`](./templates/reddit_chat_bridge.xml)
- **Image:** `ghcr.io/mmenanno/reddit_chat_bridge:latest`

---

## GasMoney

<img src="https://raw.githubusercontent.com/mmenanno/gasmoney/main/app/assets/icons/icon-1024.png" width="96" align="right" alt="GasMoney icon">

Self-hosted, single-screen calculator that estimates the gas cost of any trip you've driven (or are about to drive). Feed it GasBuddy CSV exports — or wire up the optional auto-sync that drives a bundled headless Chromium through GasBuddy's login — and it works out per-trip cost from the fuel-economy and pump-price values bracketing the trip date.

- **Project:** [mmenanno/gasmoney](https://github.com/mmenanno/gasmoney)
- **Template:** [`templates/gasmoney.xml`](./templates/gasmoney.xml)
- **Image:** `ghcr.io/mmenanno/gasmoney:latest`

---
