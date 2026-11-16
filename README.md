# Hermes Agent Recipes

Scheduled jobs and automations for [Hermes Agent](https://hermes-agent.nousresearch.com/docs) (Nous Research), written as recipes that can be recreated on any Hermes setup.

## Recipe 1: Legal AI Briefing

A daily news briefing for litigation support and e-discovery professionals. Every morning, Hermes searches the web, picks the five most significant stories from the last 24 hours, summarizes them, and delivers the result to Telegram.

| Setting | Value |
|---|---|
| Job name | Legal AI briefing |
| Schedule | `0 7 * * *` (every day at 7:00 AM) |
| Timezone | America/Los_Angeles |
| Delivery | Telegram home channel |
| Tools used | Web search and extract |

### What it covers

- E-discovery
- Legal AI
- Relativity
- Generative AI in litigation

### Requirements

- Hermes Agent installed and working with a model provider
- The Hermes gateway running (the gateway is what fires scheduled jobs)
- Telegram connected, with a home channel set (send `/sethome` to your bot)
- Web search working (test by asking Hermes to search for a current headline)

### The prompt

```
Search the web for news from the last 24 hours about e-discovery, legal AI, Relativity, and generative AI in litigation. Pick the 5 most significant stories. For each one give a bold headline, a two-sentence summary, why it matters to a litigation support professional, and the source link. Skip vendor press releases unless they announce a major product change. Keep the whole thing under 400 words.
```

Scheduled jobs run in a fresh session with no memory of earlier chats, so the prompt states everything the job needs: topics, time window, number of stories, format, and length.

### How to create it

**Option A: plain English.** Send this to Hermes in the chat or through Telegram:

```
Create a cron job named Legal AI briefing. Schedule: every day at 7:00 AM in the America/Los_Angeles timezone. Deliver to telegram. Use this exact prompt for the job: Search the web for news from the last 24 hours about e-discovery, legal AI, Relativity, and generative AI in litigation. Pick the 5 most significant stories. For each one give a bold headline, a two-sentence summary, why it matters to a litigation support professional, and the source link. Skip vendor press releases unless they announce a major product change. Keep the whole thing under 400 words. After creating it, show me the job ID and the next run time in Pacific time.
```

**Option B: command line.** Run this at the terminal prompt (not inside a Hermes chat):

```bash
hermes cron create "0 7 * * *" "Search the web for news from the last 24 hours about e-discovery, legal AI, Relativity, and generative AI in litigation. Pick the 5 most significant stories. For each one give a bold headline, a two-sentence summary, why it matters to a litigation support professional, and the source link. Skip vendor press releases unless they announce a major product change. Keep the whole thing under 400 words." --name "Legal AI briefing" --deliver telegram --timezone America/Los_Angeles
```

### How to test it

```bash
hermes cron list
hermes cron run "Legal AI briefing"
```

The briefing should arrive in Telegram within a few minutes.

### How to manage it

| Action | Command |
|---|---|
| List all jobs | `hermes cron list` |
| Run now | `hermes cron run "Legal AI briefing"` |
| Pause | `hermes cron pause "Legal AI briefing"` |
| Resume | `hermes cron resume "Legal AI briefing"` |
| Remove | `hermes cron remove "Legal AI briefing"` |
| View logs | `hermes logs` |

You can also ask Hermes in plain English, for example: "Pause the Legal AI briefing job."

### Customizing

- **Weekdays only:** change the schedule to `0 7 * * 1-5`.
- **Different topics:** replace the topic list in the first sentence of the prompt.
- **Different audience:** change "litigation support professional" to your own role.
- **Shorter or longer:** change the number of stories and the word limit.

### Lessons learned

- **Timezone.** A VPS usually runs on UTC. If the Hermes timezone is unset, "7 AM" fires at 7 AM UTC. Fix it with `hermes config set timezone America/Los_Angeles`, then restart the gateway so it picks up the change.
- **Docker.** In the Hostinger one-click Docker install, the gateway is supervised inside the container and restarts itself. `hermes gateway install` does not apply there. To bring the container back after a server reboot, set the restart policy from the host: `docker update --restart=unless-stopped <container_id>`.
- **Stopping is sticky.** If you stop the gateway on purpose, it stays stopped across restarts until you run `hermes gateway start`.

### Security notes

- Never commit your `.env` file. It holds your Telegram bot token and API keys.
- Restrict the Telegram bot to your own user ID (`TELEGRAM_ALLOWED_USERS`). Anyone on that list can give commands to an agent that has access to your server.
- This recipe contains no secrets and is safe to share.

### Tested on

- Hermes Agent, Hostinger VPS (one-click Docker install)
- Model: DeepSeek
- November 2026
