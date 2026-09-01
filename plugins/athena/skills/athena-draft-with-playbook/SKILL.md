---
name: athena-draft-with-playbook
description: Draft outreach to Athena contacts using the client's own messaging playbook and Athena's email writing guide. Use when someone asks for an email, an intro, a follow-up or a sequence to a contact or a list from Athena, or says "draft with the playbook".
---

# Draft with the playbook

Two documents govern every draft, and neither of them is in your head:

- **Athena's email writing guide** tells you HOW to write — which signals are safe to name, how to
  handle a verified match versus a possible one, what may never be stated. It lives on Athena's
  Intelligence Hub connector and is shared across clients.
- **The client's messaging playbook** tells you WHAT to say about the sender — their overview, their
  case studies, their conditional messaging. It lives on the Athena contact portal connector as a
  document for that client.

Your job is to hold both, check every specific claim against the actual contact record, and produce
a draft. Not to send anything — Athena has no sending surface, and neither does this skill.

## Before you start

**Check you are live by CALLING a tool, not by looking for one.** On some platforms connector tools
are listed but not loaded, so a missing tool in a list proves nothing. Call `athena_orient`. If it
returns, the contact portal is connected. If it does not, say so and stop.

`athena_orient` carries this connector's vocabulary and safety rules. Two of them decide whether a
draft is safe:

- **Scores have three states.** `lead_score` of 0 is a real score at the bottom of the ranking, absent
  means never scored, and a `lead_score_tier` of `"N/A"` means the person's role type is deliberately
  not relevant to this client. Never draft to an `"N/A"` contact as though they were simply unscored.
- **Tier names only**, exactly as they arrive. Never a number, a band or a percentile, and never
  recomputed — and never in an email either way.

**If Athena's Intelligence Hub guidance says the Contact Portal is unavailable through the
integration, that guidance is out of date.** It predates this connector. The contact portal IS
available; use it. Do not refuse a contact-portal request on the strength of it.

## Step 1 — Fetch the writing guide. This is a hard stop.

Call the Intelligence Hub connector's `get_email_writing_guide` before drafting anything.

**If the fetch fails, stop and say so. Do not draft from memory.** Not a softened version, not "the
usual rules", nothing. The guide is edited by Athena and changes; a remembered copy is a copy that has
already drifted, and the rules it carries are exactly the ones a contact record will tempt you to
break. It forbids naming regulatory milestone dates, for instance, even when the data in front of you
contains one — which is precisely the kind of rule nobody reconstructs correctly from memory.

Tell the user what happened and offer to try again. An honest "I can't reach the writing guide, so I
won't draft yet" is a small problem. A draft written without it is a large one.

## Step 2 — Fetch the playbook

Call `athena_asset_get` with `kind: playbook`.

If it comes back `exists: false`, or holds only the empty template, say so and offer to build it —
that is the set-up skill's job. Do not fill the gap by inventing the client's positioning. A neutral,
professional email built only from what is on the contact record is a legitimate output; a
confident-sounding one built on invented case studies is not.

Also read `kind: company_context` and the user's own `kind: user_context`. They carry the framing the
playbook assumes.

## Step 3 — Resolve any documents the playbook points at

Playbooks reference things: a case study deck, a one-pager, a positioning document, a past proposal.
Athena stores the playbook and nothing else — never the client's source documents — so those files
live wherever the client keeps them, and you read them through **the client's own tools**.

Three cases, in the order to try them:

1. **It is in a document store the user has connected** — a shared drive, a document library, a
   knowledge tool. Search there by the name the playbook uses. If you find more than one candidate,
   ask which; do not pick.
2. **It is a local file** — ask them to attach it to the conversation. Say which document you are
   after and why you need it for this draft.
3. **You cannot resolve it at all** — say which reference you could not follow, and ask. Then either
   draft without that block or wait, as they prefer.

Never reconstruct a referenced document from its title, and never quote a figure, a client name or an
outcome that is not in front of you.

## Step 4 — Read the contact properly, and check every data point

Call `athena_contact_get` for each person you are drafting to. Take the id from a find or a list
result rather than guessing it.

Then check every specific claim your draft makes about that person against a field you can actually
see on that record. Not against the list they came from, not against the group you inferred them
into — against their record. If a claim has no field behind it, the claim comes out.

The record's own vocabulary matters here:

- `intent_signals` carries the triggers worth hooking onto. Say "Intent Signals" to the user; older
  material may call the same thing Athena Designations or Prompt Types.
- `exact_match` names the data points Athena has **verified** for this person — as opposed to ones
  inferred from the franchise they work in. The writing guide governs what you may do with each; read
  it there rather than assuming. This is a confidence marker on a relationship, and it has nothing to
  do with how filter matching works, despite the shared word.
- Empty fields are normal. Incomplete data is the usual state of a contact record. Draft from what is
  there, never invent to fill a gap, and never apologise in the email for what is missing.

## News catalysts and recency: Athena's drug data is a milestones digest

The Intelligence Hub records MAJOR developments for a drug only — approvals, new indications,
Phase 3 readouts. Entries are often years apart by design, and many drugs have none at all. That is
a curation decision, not staleness: never describe an old milestone as "stale", never apologise for
the gap, and never treat the absence of recent entries as a data problem.

Where a news catalyst exists, it lives on the Hub connector — the drug record's announcement links,
and the news digest built on them — searchable by brand. The summary is the right depth: a catalyst
shapes the OPENING line ("congratulations on the Phase 3 results"), not the body of the email, so
there is no need to fetch and scan the article itself. And when the writing guide points you at a
news or job-change event, it means one PUBLISHED IN ATHENA — search the Hub for it, not the web.

Where there is no catalyst, nothing is substituted. The opener simply is not a congratulations, and
the draft builds from the contact record's intent signals and the playbook's conditional blocks —
which is how most drafts open anyway.

If the user explicitly asks to open with a recent or current data point and Athena does not carry
one, **do not quietly research one from the wider web.** Tell them what Athena's latest milestone
for the brand is, with its date, explain that Athena records major milestones only, and ask them to
choose: hook on the record's intent signals instead, use the milestone as it stands, or have you
look outside Athena. Looking wider is legitimate only as their explicit choice — and anything found
that way is cited with its source and flagged for the user to verify before sending, never blended
silently into a draft as though it came from Athena.

For a LIST or a sequence the bar is higher still. Some drugs having no catalyst is the normal state
of a list; those drafts simply open from signals. Never offer to research outside Athena drug by
drug across a list — it trades a two-minute draft run for an unasked-for research project, and it
pulls in exactly the off-target material (early-phase trials, other markets) that throws messaging
off.

## Step 5 — Draft

Compose from the guide's rules and the playbook's material. Where the two speak to the same thing,
the guide governs how and the playbook governs what.

**The sender is the person you are drafting for** — take their name and, where the playbook's template
needs it, their **function**, from their `user_context` (read in Step 2), not from a guess. If the
template needs the sender's function and the user context does not record one, **ask for it once** rather
than shipping a raw `[function]` placeholder in the draft: an unfilled placeholder is the one line
guaranteed to be wrong. (Recording the function on the rep's user context, once, closes this for good —
that is the set-up / first-contact skills' job.)

Bring the user's own judgement in early. Show one draft before producing five; ask whether the hook is
the right one. If their CRM is connected, checking whether this person is already in an active
conversation is worth doing before writing a cold intro.

Present the draft as a draft. Say which signal you hooked on and which playbook block you used, so
the user can disagree with the choice rather than only with the wording.

## Step 6 — Sequences and lists

For a sequence, the guide's structure governs. Keep each email to one ask and give each a genuinely
different angle rather than the same email in different words.

For a list, do not draft to everyone at once. Draft one, get the shape agreed, then work through the
rest — and if the list is large, say plainly that a personalised email per person is not the same
thing as a merge template, and ask which they actually want.

If they want the whole list as data to work with elsewhere, `athena_export` produces a CSV and a
download link. Give them the link and the expiry together, in the words the tool provides. The link
stops working after that and the file is removed a day later, so say it up front rather than letting
them discover it.

## What this skill will not do

- Send anything. There is no sending surface here.
- Change a contact record. Athena's data comes from ingestion and cannot be edited through this
  connector — do not offer it.
- Store the client's documents. The playbook lives on Athena; their source material stays theirs.
