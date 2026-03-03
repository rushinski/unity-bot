# Unity Bot - Community Engagement & Support

[![Live Bot](https://img.shields.io/badge/Live-Active-green?logo=discord)](https://discord.com/)  
A **modular Discord bot** built to consolidate commands, events, and support workflows into **one scalable system**.
Originally designed for a **large-scale gaming community (Kingdom 3743, ~900 members)**.

Explore the **[Unity Bot Landing Page](https://rushinski.github.io/Unity-Landing-Page)** for a live showcase.

---

## Live Usage

- **Primary Server:** *Kingdom 3743* (~900 members, est. May 2024)  
- **Repo Owner:** [rushinski](https://github.com/rushinski)

---

## Features

### Community Engagement
- **Leveling & Leaderboards** → gamified chat progression, message-based leveling engine, `/leaderboard` and `/levelprogress` commands.
- **Ticketing System** → dropdown ticket creation, modal-based input, role-based verification, support pings with cooldown, closure transcripts (stored in GitHub Gist, fallback to MongoDB).  
  - **1000+ transcripts archived** across servers.
- **Role Selection & Counts** → `/sendRolesSelect` for self-assignable roles, reaction role categories, and **real-time role count voice channels**.

### Moderation & Security
- **Rule Enforcement** → banned words filter with fuzzy matching + severity tiers (warn, strike, auto-ban).
- **Moderation Tools** → `/idBan`, `/idUnban`, `/strike`, `/verifyUser`, `/clear` with auto-escalation (3 strikes = ban).
- **Infractions Tracking** → Persistent tracking in MongoDB with automated resets after bans.
- **Verification System** → onboarding tickets with manual/automatic verification workflows.

### Engagement Utilities
- **Giveaways** → `/sendGiveawayMessage` with persistent schema for entrants/winners, resumes after restart.
- **Utilities** → `/getUtc`, `/say`, scheduled UTC-ready channels via `readyUtc` event.

### Infrastructure & Extensibility
- **Dynamic Loaders** → auto-registration of commands, events, and UI components.
- **Access Control** → flag-driven restrictions (`admin`, `owner`), cooldowns, and Discord-native permission checks.
- **Hosting/Deployment** → optimized for Discloud + VPS hosting with backups and snapshots.
- **Persistence** → MongoDB models for users, configs, tickets, transcripts, infractions, giveaways, and role systems.
- **External Integrations** → GitHub Gist for transcript archiving, GitHub Pages for landing page.

---

## Impact

**893+ community members** in Kingdom 3743  
**8 servers** actively running the bot  
**1000+ support transcripts archived**  
**3 paid bot development offers** generated from this project  
**Scaled hosting** → upgraded from free-tier to paid server due to growth  
**Security-aligned** → built to integrate with Discord’s security & permission model

---

## Tech Stack

**Bot Core**
- [Node.js](https://nodejs.org/) + [discord.js](https://discord.js.org/) v14
- MongoDB + Mongoose (persistence)

**Features**
- Slash Command Framework (`SlashCommandBuilder`)
- Modal-based ticket & verification flows
- Gamification/leveling engine with persistence
- Giveaway engine with auto-resume
- GitHub Gist integration for transcripts
- Reaction roles + role-count channels

**Infrastructure**
- Hosting: Paid VPS + Discloud deployment  
- Config: `config.json` (tokens, Mongo URL, bot ID) + `.env` secrets  
- Security: Discord permission model + flag-based command access

---

## Repository Structure

```text
bot/
├── commands/              # Slash command implementations
│   ├── configurations/    # Guild configs (tickets, roles, logs, UTC)
│   ├── levels/            # Leveling commands (leaderboard, add/remove/reset)
│   ├── moderation/        # Moderation (ban/unban, strikes, verifyUser, clear)
│   ├── misc/              # Misc commands (getUtc, say)
│   └── send/              # Setup messages (tickets, verification, roles, rules, giveaways)
│
├── events/                # Event handlers (tickets, moderation, logging, roles, UTC)
├── schemas/               # MongoDB models (users, infractions, tickets, transcripts, configs, giveaways, roles)
├── utils/                 # Infrastructure + helpers (loaders, level utils, ticket creation, gist)
├── data/                  # Static data (levels, banned words)
├── modals/                # Modal handlers (e.g., /say input)
│
├── index.js               # Entrypoint, client bootstrap + loaders
├── config.json            # Bot config (tokens, Mongo URL, bot ID)
├── package.json           # Dependencies
└── package-lock.json      # Lockfile
```

---

## Reliability & Testing

### Testing & QA
- All testing conducted in a **private Discord server** prior to deployment.
- Error recovery workflow:
  - Monitor **Discloud logs** via mobile.
  - Restart bot quickly if errors occur.
  - Debug and patch fixes directly in **VS Code**.

### Error Handling
- `try/catch` used across commands/events.
- **Central handling** in `index.js`.
- **Utils-level handling** (all except `levelUtils.js`).
- `EventLoader.js` ensures robust recovery for failed module loads.

---

## 📖 Additional Documentation

- [ARCHITECTURE.md](./docs/ARCHITECTURE.md) → System design and lifecycle flow  
- [INTEGRATIONS.md](./docs/INTEGRATIONS.md) → Discord API, MongoDB, GitHub Gist integrations  
- [SECURITY.md](./docs/SECURITY.md) → Role-based restrictions, cooldown enforcement, data handling

---

## Future Work

### Scaling & Extensibility
- Add moderation commands: `/unstrike`, `/warn`, `/unwarn`, `/timeout`, `/untimeout`, `/kick`, `/set-banned-words`.
- Owner-only commands for scalability: `/reload`, `/shutdown`, `/deploy`.
- Combine schemas for efficiency.
- Unified config management (removes + lists in same command).
- Expanded channel logging capabilities.
- Leaderboard pages & searchability to show ranking beyond top 10.
- Configurable strike system.
- Multi-ticket configurations.

---

Portfolio Case Study: This bot demonstrates **scalable engineering for real-world community management**, bridging moderation, engagement, and structured support into one unified system.
