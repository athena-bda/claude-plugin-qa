# Athena BDA — Claude plugin

Athena's plugin for Claude: the skills Athena's own team uses to work with the **Athena Contact
Portal** and the **Athena Intelligence Hub**, packaged for one-step installation.

## Installing

In Claude Code:

```
/plugin marketplace add athena-bda/claude-plugin-qa
/plugin install athena@athena-bda
```

On claude.ai, add this repository as a plugin marketplace from the plugins settings, then install
the **athena** plugin from it.

## Connect the two Athena connectors

Installing the plugin gives you the skills. On some surfaces (including claude.ai on the web) the
bundled connector declarations are not applied automatically, so add both connectors yourself as
custom connectors, and sign in to each:

- **Athena Contact Portal** — `https://athena-qa-api.azurewebsites.net/mcp`
- **Athena Intelligence Hub** — `https://vbwrebrltrrvijaslhes.supabase.co/functions/v1/mcp`

Sign-in uses your normal Athena account. If a tool call answers that access is not enabled, your
company has not been released onto the connector yet — the portal itself is unaffected; contact
Athena.

## The skills

| Skill | What it does |
| --- | --- |
| `athena-set-up` | Set up or tune a company on Athena: company context, each person's patch, and the messaging playbook |
| `athena-ask-the-data` | Turn a plain-language request into a counted, saved list or view, or an export |
| `athena-whats-next` | The "What's next?" briefing — what has changed since you last looked |
| `athena-draft-with-playbook` | Draft outreach from your playbook and Athena's email writing guide |
| `athena-conference-play` | Turn a conference into a working list of people worth meeting |
| `athena-rep-first-contact` | A rep's first session: read back what was set up, save their standing views |

## Support

support@athenabda.com
