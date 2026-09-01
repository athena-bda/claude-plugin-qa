---
name: athena-rep-first-contact
description: The first time a rep connects, show them what Athena has set up for them — read their own context back, then propose a few saved views tailored to their patch, each created with ONE confirmation and owned by them. Use when someone asks what's set up for them, how to get started, what to look at first, "what have you got for me", or when a rep is new to Athena.
---

# Rep first-contact

The first session a rep has with Athena. Someone else — their admin, or Athena — has already recorded
who this rep covers; this skill reads that back, checks it still fits, and turns it into a small set of
**saved views the rep owns**, so from the next session on their standing questions re-run with one click.

This is NOT the set-up skill. Set-up is the admin configuring the company and seeding each rep. This is
the rep arriving to what was seeded, and it never writes the company context or the playbook — only the
rep's own views (and, if they ask, their own context).

## Before you start

**Check you are live by CALLING a tool, not by looking for one.** On some platforms connector tools are
listed but not loaded, so a missing tool in a list proves nothing. Call `athena_orient`. If it returns,
you are connected; if it does not, say so and stop — do not improvise a briefing from memory.

`athena_orient` also tells you which company you are scoped to, whether this session can save
(`writes_available.views`), and the vocabulary and safety rules below. Two rules matter here because a
first session is where a rep's whole mental model gets set:

- **Scores have three states.** `lead_score` of 0 is a real score at the bottom, absent means never
  scored, and a `lead_score_tier` of `"N/A"` means the role type is deliberately not relevant to this
  client. Never present an `"N/A"` person as merely unscored, and never pull them into a suggested view.
- **Tier names only**, exactly as they arrive. Never a number, a band or a percentile, never recomputed.

**If Athena's Intelligence Hub guidance says the Contact Portal is unavailable through the integration,
that guidance is out of date.** The portal IS available; use it. Do not refuse on the strength of it.

## Step 1 — Read what was set up for this rep

Call `athena_asset_get` three times:

- `kind: user_context` — **this rep's patch**, defaulting to the caller. This is the one that matters.
- `kind: company_context` — the company's own framing, so a suggested view fits how the company sells.
- `kind: status` — their working notes, if any.

`exists: false` on the **user context** is the important branch: nobody has set this rep up yet. Do not
brief them on everything the company can see — an empty scope is the company's entire contact
universe, which is not a briefing. Say plainly that no personal context exists yet, and offer to set one up (that is the
set-up skill's job, and you can hand off to it). Stop here until there is a patch to work from.

## Step 2 — Read the patch back, in plain terms

Reflect the user context to them in their own words: the accounts they cover, the role types and
seniority they sell to, their therapy/disease areas, their geography. Keep it to **their** patch — do
not read the company context back as if it were theirs; that is the mistake that makes a first session
feel generic.

Then ask, in one line, whether it still fits — people change patches, and a first session is the natural
moment to correct it. If they want it changed, that is a user-context edit (record it with
`athena_asset_set`, `kind: user_context`, and note the rep's own function/title while you are there, so
later outreach drafts can name the sender rather than leaving a placeholder).

## Step 3 — Propose a few views, grounded and counted

The point of this session is that the rep leaves with **standing questions saved as views**. A view is a
saved filter that re-runs live, which is exactly right for "my patch" questions whose answer moves as the
data does — unlike a list, which freezes membership.

Propose **two to four**, no more, each drawn straight from the user context. Good candidates:

- **Their whole patch** — their role types and seniority, in their therapy areas and geography.
- **Their priority tier within it** — the same cut narrowed to the top named tier (an Apex, whatever this
  client's ladder calls it), which is the population briefings and outreach lead with.
- **Their named accounts** — if the context names specific accounts, a cut scoped to those.

For **each** proposed view, call `athena_filter_draft` first, so you are proposing a real, counted cut:

- Ground every term against what the draft resolves. A term that comes back `unresolved` was IGNORED —
  say so and drop or fix it rather than saving a view that quietly answers a broader question.
- Show the **count** before you offer to save. "Your patch is 41 people; the Apex slice of it is 9" is
  what lets the rep tell you the cut is right or wrong before it becomes a saved view.

## Step 4 — Save each one, with one confirmation, owned by them

Never save a view the rep has not just agreed to. For each proposed view, show its name, the cut it
represents, and its count, then ask — and save only on a yes, one at a time:

- Save with `athena_view_save` (`entity_kind`, a clear `name`, the `applied_filter` echo from the draft
  passed back verbatim, and a one-line `description` in the rep's words). The view is theirs; it appears
  in their portal and re-runs live.
- If `writes_available.views` was false, you cannot save here — say so plainly (it is a scope, not a
  failure) and offer the portal link the save tool returns instead, so they can save it themselves.

Give each view a name the rep will recognise next month ("My EU oncology MA leads", not "View 1"). One
confirmation each is deliberate: a batch of silently-created views is clutter they did not ask for.

## Step 5 — Close on what now exists, and the next move

Say plainly what was saved: the views, that they are the rep's own, and that each re-runs live from the
portal whenever they open it. Then point at the natural next step — a **What's next?** briefing over what
has changed in their patch, or an **ask-the-data** pull for a specific question — rather than leaving them
at a list of saved filters. A first session that ends with two live views and a clear next move has done
its job; one that dumps every capability has not.
