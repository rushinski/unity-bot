# Unity Bot - System Architecture

## Purpose

Unity Bot is a **modular, event-driven Discord bot** built with **Node.js** and **discord.js v14**, designed around **subsystems** that scale independently while sharing a common persistence layer in **MongoDB**.

It has been deployed in production Discord servers, powering moderation, community engagement, and automation workflows.

---

## 🌐 System-Wide Overview

```mermaid
flowchart TD
    U[Discord User] -->|Commands, Messages, Reactions| D[Discord API]
    D --> B[Bot Client]

    B --> T[Ticketing & Verification]
    B --> M[Moderation]
    B --> L[Leveling System]
    B --> R[Roles & Counts]
    B --> G[Giveaways]
    B --> C[Configuration]

    T --> DB[(MongoDB)]
    M --> DB
    L --> DB
    R --> DB
    G --> DB
    C --> DB

    T --> GIST[GitHub Gist]
```

---

## Entrypoint

* **`index.js`** bootstraps the bot.

  * Initializes the Discord client with required intents.
  * Connects to MongoDB.
  * Loads commands, events, and modals via loaders in `utils/`.
  * Registers slash commands with Discord’s API.
  * Routes interactions to the appropriate handler.

```mermaid
flowchart TD
    A[Start Bot] --> B[Load Config]
    B --> C[Connect MongoDB]
    C --> D[Load Commands]
    C --> E[Load Events]
    C --> F[Load Modals]
    D --> G[Register Slash Commands]
    E --> G
    F --> G
    G --> H[Bot Ready]
    H --> I[Active Operations]
```

---

## Subsystems

### Ticketing & Verification

* **Flow**: Dropdown → modal → ticket channel → support/verification → close → transcript archive.
* **Features**:

  * Configurable ticket categories per guild.
  * Verification tickets for onboarding.
  * Buttons to ping staff (with cooldown), verify users, close tickets.
  * Closed tickets archived with transcripts.
* **Persistence**: `Ticket`, `TicketTranscript` schemas.
* **Transcripts**: Stored via GitHub Gist integration, fallback to MongoDB.
* **Impact**: 1000+ transcripts archived across production servers.

```mermaid
flowchart TD
    A[User Selects Ticket Type] --> B[Modal Input Submitted]
    B --> C[Private Ticket Channel Created]
    C --> D[Support Team/Verification Workflow]
    D --> E[Ticket Closed by Staff]
    E --> F[Transcript Generated]
    F --> G{Storage Option}
    G -->|Primary| H[GitHub Gist]
    G -->|Fallback| I[MongoDB Transcript Collection]
```

---

### Moderation

* **Commands**: `/ban`, `/unban`, `/idBan`, `/idUnban`, `/strike`, `/verifyUser`, `/clear`.
* **Events**: `checkBannedWords`, `messageDelete`, `messageUpdate`.
* **Escalation Policy**:

  * Low severity → warning.
  * Medium severity → strike.
  * High severity → 2 strikes.
  * Critical severity → immediate ban.
* **Persistence**: `Infractions` schema.
* **Transparency**: Logged actions, auto-reset infractions post-ban.
* **Rate-Limiting**: Cooldowns and Discord permission checks prevent abuse.

```mermaid
flowchart TD
    A[Message Sent] --> B{Contains Banned Word?}
    B -->|No| C[Allow Message]
    B -->|Low Severity| D[Warn User]
    D --> E[Check Total Strikes]
    B -->|Medium Severity| F[Issue Strike]
    F --> E
    B -->|High Severity| G[Issue 2 Strikes]
    G --> E
    B -->|Critical Severity| H[Immediate Ban]
    E --> I{Strikes >= 3?}
    I -->|Yes| H
    I -->|No| J[Record Infraction in DB]
```

---

### Leveling System

* **Tracking**: Each message increments XP (`messageCreate` event).
* **Features**:

  * Level thresholds defined in `data/levels.js`.
  * `/leaderboard` shows top 10 members.
  * `/levelProgress` shows current progress toward next level.
  * Admin overrides: `/addMessages`, `/removeMessages`, `/resetAllMessages`.
  * Opt-in/out of level-up notifications.
* **Persistence**: `UserSchema`.
* **Utils**: `levelUtils.js` centralizes XP → level conversion.

```mermaid
flowchart TD
    A[User Sends Message] --> B[Increment Message Count]
    B --> C[Recalculate Level]
    C --> D{Level Up?}
    D -->|Yes| E[Update User Record + Notify]
    D -->|No| F[Continue Tracking]
```

---

### Roles & Counts

* **Reaction Roles**:

  * Users add/remove reactions on role-select embeds.
  * Automatically grants or removes roles.
  * Stored in `RoleReactionMessage`.
* **Role Counts**:

  * Voice channels dynamically renamed to show live member counts.
  * Config stored in `RoleCountConfig`.

```mermaid
flowchart TD
    A[User Reacts to Role Message] --> B[Match Emoji to Role]
    B --> C[Assign Role]
    A2[User Removes Reaction] --> D[Remove Role]
    E[Role Count Config] --> F[Update Voice Channel Name]
```

---

### Giveaways

* **Commands**: `/sendGiveawayMessage`.
* **Persistence**: `Giveaway` schema (title, prize, winners, end time).
* **Lifecycle**:

  * Giveaway started → DB record created.
  * Participants added during runtime.
  * On scheduled end → winners selected automatically.
  * Giveaway state restored on restart.

```mermaid
flowchart TD
    A[Start Giveaway Command] --> B[Save Giveaway in DB]
    B --> C[Collect Entrants]
    C --> D[Wait Until End Time]
    D --> E[Select Winners]
    E --> F[Announce Results]
```

---

### Logging & Lifecycle

* **Guild Events**:

  * `guildMemberAdd` → welcome embed, join log, update member count.
  * `guildMemberRemove` → departure log, update member count.
* **Message Logs**:

  * `messageDelete` → logs deleted content.
  * `messageUpdate` → logs old vs new content.
* **Startup Recovery**:

  * `ready` restores giveaways and reaction roles.
  * `readyUtc` updates configured UTC channels.

```mermaid
flowchart TD
    A[Bot Startup] --> B[Restore Giveaways]
    A --> C[Restore Reaction Roles]
    A --> D[Set Presence]
    A --> E[Update UTC Channels]
```

---

## Persistence Layer

* **Guild Configs**: Ticket categories, log channels, role counts, reaction roles.
* **Users**: Levels, XP, message counts, notification preferences.
* **Infractions**: Strikes, warnings, bans.
* **Tickets & Transcripts**: Active tickets + archived support history.
* **Giveaways**: Entrants, winners, metadata.
* **Role Systems**: Reaction roles, role count configs.

---

## External Integrations

* **Discord API** → Slash commands, modals, embeds, events.
* **MongoDB** → Persistent datastore for all systems.
* **GitHub Gist** → Ticket transcripts.
* **discord-html-transcripts** → Transcript formatting.
* **date-fns / node-cron** → UTC scheduling + cron jobs.
* **fast-levenshtein** → Fuzzy matching for banned words.
* **GitHub Pages** → Public landing page.

---

## Strengths

* Modular subsystems with clear separation of concerns.
* Recovery after restarts (reaction roles, giveaways).
* Configurable per guild.
* Transparent logs for moderation and support.
* Full audit trail with transcripts and infractions.

---

## Future Work

* Moderation: `/unstrike`, `/warn`, `/unwarn`, `/timeout`, `/untimeout`, `/kick`, `/set-banned-words`.
* Owner-only commands: `/reload`, `/shutdown`, `/deploy`.
* Consolidate schemas for efficiency.
* Unified config management (removes + list in same command).
* Expanded channel logging.
* Leaderboard pagination + searchability.
* Configurable strike escalation policies.
* Multi-ticket configurations.
* Automated penetration testing & anomaly detection.

---
