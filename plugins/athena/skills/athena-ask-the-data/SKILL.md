---
name: athena-ask-the-data
description: Answer a plain-language request against Athena's data — find people and accounts, turn a description into a counted, saved list or view, and export it. Use when someone wants to pull, filter, look up, count, build, refine, save or export contacts, accounts, lists or views ("show me…", "who are our…", "build me a list of…", "how many…", "export that").
---

# Ask the data

The pull conversation. Someone describes the cut of the world they want — "our high-quality oncology
contacts in the EU", "who runs medical affairs at these ten accounts", "everyone we haven't touched
since June" — and this turns it into something real: a count they can trust, a list or view they own,
a file they can take away. The value is in doing it *honestly* — grounding every word against what the
data actually contains, and confirming with a count before anything is saved or exported.

## Before you start

**Check you are live by CALLING a tool, not by looking for one.** On some platforms connector tools
are listed but not loaded, so "I can see the tools" proves nothing. Call `athena_orient`. If it
returns, you are connected and it tells you the company you are scoped to and whether writes are
available. If it does not return, say so and stop — never answer from memory or from an earlier
conversation.

`athena_orient` also carries this connector's vocabulary and safety rules. Two of them get broken
exactly here, in the pull, so they are worth repeating:

- **Scores have three states, and one number.** `lead_score_standardized` is the only lead score you
  will see or say. `lead_score_standardized` absent means not scored on this client's standardised
  scale. `lead_score_standardized` 0 with a tier is a REAL score, at the bottom of the ranking.
  `lead_score_standardized` 0 with `lead_score_tier` "N/A" means one of this client's own scoring
  rules ruled the person out: a value they marked as unwanted, with nothing scored to outweigh it.
  Say "ruled out by your scoring rules", never "unscored", and keep them out of priority lists. A
  ruled-out person's score is 0 too, so the TIER is the only thing that tells the two apart. N/A on a
  data field — therapy area, remit, and so on — is a different thing entirely: it means Athena has no
  information. Say "unknown", and never exclude anyone on it. Never fold absent into zero.
- **Tier names only**, exactly as they arrive. Never turn one into a number, a band or a percentile,
  and never recompute a score. A tier is a LABEL, never a filter of its own: prioritise by
  `lead_score_standardized`, present the tier name beside it, and never make a tier the sole reason
  to include or leave someone out. "Highest priority" means the top of the score ordering, not one
  named tier. When someone asks for a count, walk down the score ordering until you have that many
  and say each one's tier as you go — a High at 51 and a Medium at 50 are neighbours.

**Use the user's other tools.** If their CRM or calendar is connected, use it — checking whether a
name is already an open opportunity, or already met, makes the answer better. Athena is not trying to
be the only thing in the room.

## Step 1 — Ground the ask in the real vocabulary

Before turning a description into a filter, find out what words the data actually uses. Call
`athena_filter_options_get` and read the live facets — the tiers that exist, the intent signals, the
geographies, the role types, the connection types. Map the user's phrasing onto those, and do not
invent a value that is not there.

"High-quality" is not a field, and it is not one named tier either. It resolves to the **top of the
score ordering**: order by `lead_score_standardized`, say each person's tier name beside their score,
and honour a requested count by walking down that ordering until you have it. Cutting at a tier
boundary drops the people just outside it, who are usually exactly who was meant. If the ask is
genuinely ambiguous — a phrase that could mean two different cuts — say what you found and let the
user pick, rather than guessing.

## Step 2 — Draft the cut, and COUNT before you commit

Call `athena_filter_draft` to resolve the terms and get a **count** back. This is the moment that
keeps the pull honest, so do not skip it:

- **Report what did not resolve.** If a term came back unresolved or ambiguous, say so plainly. A cut
  that silently dropped half the request is worse than one that asks a question.
- **Show the count and let it steer.** "That's 4,120 people — wider than a campaign usually wants.
  Shall we tighten to the EU and the top tier?" A count the user reacts to is how a good list gets
  built; refine and re-count until the shape is right.
- **Never describe a saved or exported result you have not actually counted.** The number comes from
  the tool, not from an estimate.

## Step 3 — Look-ups

When the ask is about a specific person or account rather than a set — "tell me about this contact",
"who are the medical affairs leaders at this account" — use `athena_contact_find` / `athena_contact_get`
and `athena_account_find` / `athena_account_get`. Read the fields off the record; do not embellish. If a
field is absent, it is absent — that is information, not a prompt to fill it in.

## Step 4 — Save it, as the right kind of thing

Only save when the user wants to keep it, and know which of two things they are asking for — they
behave differently and the user should know which they got:

- **A list** is a fixed set of members captured now. Build it with `athena_list_create`, then adjust
  membership with `athena_list_members_add` / `athena_list_members_remove`. Use a list for a campaign
  cohort — "these 300 people, this quarter" — where the membership should not drift underneath them.
- **A view** is a saved filter that re-runs. Save it with `athena_view_save`. Use a view for a standing
  question — "our Apex EU oncology contacts" — where they want the *current* answer each time they open
  it, not a frozen snapshot.

Say which one you created, and where it now lives, so the user can find it in the portal.

Writes are not always available — `athena_orient` said whether they are. If a scope is read-only,
decline the save cleanly and say why (it is a scope, not a failure); do not pretend it saved.

## Step 5 — Export

When the user wants to take the data away, call `athena_export`. It returns a **time-limited download
link**, not an inline dump — hand the link over as it comes, note that it expires, and do not paste the
rows into the conversation. The file is scoped to the same data the user can already see.

## What good looks like

A pull that went well: every word in the request was grounded against a real facet or reported as
unresolved; the user saw a count before anything was saved; they know whether they got a list or a
view and where it is; and if they exported, they have a working link and know it expires. Nothing was
invented, no score was recomputed, and no "N/A" person was passed off as unscored.
