---
name: athena-set-up
description: Set up or tune a client on Athena — record who the company sells to, who each person covers, and the messaging playbook they draft from, so every later Athena conversation is already scoped. Use when someone says "set me up", "set up Athena", "tune my context", "help me build my messaging playbook", or when a briefing comes back too broad because nobody has said what the user actually cares about.
---

# Set up / tune

Athena works from four standing documents. This skill writes them:

- **Company context** — who this company is, who they sell to, what they care about. Everyone in the
  company reads it.
- **User context** — one person's patch: their accounts, their role types, what they are working on.
- **Playbook** — the messaging playbook outreach is drafted against.
- **Status note** — an assistant's own working notes for one person, including where their last
  briefing got to.

Get these right once and every later conversation starts scoped. Get them wrong and every briefing
answers a broader question than anyone asked.

## Before you start

**Check you are live by CALLING a tool, not by looking for one.** On some platforms connector tools
are listed but not loaded, so "I can see the tools" is not evidence and "I cannot see them" is not a
failure. Call `athena_orient`. If it returns, you are connected. If it does not, say so plainly and
stop — do not improvise from memory.

`athena_orient` also tells you three things this skill needs: which company you are in, whether this
session can save anything (`writes_available`), and the vocabulary and safety rules that apply to
everything below. Read them; they are not repeated here.

Two rules are worth repeating here, because a set-up conversation is where they get built into
everything that follows:

- **Scores have three states.** `lead_score` of 0 is a real score at the bottom of the ranking, absent
  means never scored, and a `lead_score_tier` of `"N/A"` means the person's role type is deliberately
  not relevant to this client. Never treat `"N/A"` as missing data and pull those people back in.
- **Tier names only**, exactly as they arrive. Never turn one into a number, a band or a percentile,
  and never recompute a score.

**Use the user's other tools.** If they have their CRM, their drive or a research tool connected, use
them. Athena is one source among several and works better beside the rest.

## Step 1 — Work out which of the three set-ups this is

Read `you` and `writes_available` from `athena_orient`.

- **`is_auto_scoped` is true** — the user belongs to one company and everything scopes to it. This is
  a client admin setting their own company up, or doing it on a call with Athena. Do not ask for a
  company id; they have one and it is already applied.
- **`is_auto_scoped` is false** — the user works across companies. They are an Athena operator, or a
  client user who belongs to more than one company. Call `athena_company_list`, show them the
  companies it returns, and ask which client this session is for. Pass that `company_id` on **every**
  subsequent call. Do not guess an id and do not carry one over from an earlier conversation.
- **Whether you can save the documents** — the four documents are published with `athena_asset_set`,
  so the write family that matters here is **`writes_available.assets`**, not `lists` or `views`. Read
  it as three cases, and do not collapse them:
  - **`is_determined` is false** — you have not scoped a company yet (you are an operator, or a
    multi-company user). This is **not** a refusal. Choose a company first (the bullet above), and let
    the company-scoped tools answer: `athena_asset_get` returns `can_edit` per document. Do not tell
    the user you cannot save on the strength of an undetermined answer.
  - **`is_determined` is true and `assets` is false** — you genuinely cannot publish these documents
    from this connection. Say so at the START, before the interview, not after it. Run the interview
    anyway if the user wants — the answers are still useful — but tell them the documents will need
    saving from a connection that can write, and do not end the session implying anything was recorded.
  - **`is_determined` is true and `assets` is true** — you can publish. Proceed.

If someone from Athena is running this for a client, everything below is identical. The documents land
in the client's company, and the people in that company read them from their next session.

## Step 2 — Ground the vocabulary before you ask about it

Call `athena_filter_options_get` for `contact`, and again for `account` if the conversation is going
to reach accounts.

This is not a formality. The values are live and differ per company: a therapy area or tier that
exists for one client may not exist for another, and a term the user says confidently may match
nothing here. Read each facet's flags as well as its values — a facet reporting `is_empty` is a
question not worth asking, and a facet reporting `is_truncated` is a sample you must not treat as the
whole vocabulary.

Two facets are worth reading before the interview even starts:

- `lead_score_tiers` — if this client's scoring is already set up, the tier names tell you so.
- `connections` — if it is empty, this client has no LinkedIn connection data, and any plan that
  leans on "who do we already know" has nothing behind it. Find that out now, not in a briefing.

## Step 3 — If scores already exist, start from them rather than from a blank page

Where a client has already built their lead scoring, the scoring IS a statement of who they care
about, and it is a much better starting point than an open question.

Draft a count per tier with `athena_filter_draft` and put the numbers in front of them: "your scoring
already puts N people in the top tier, mostly in these therapy areas — is that the population you
want briefings about?" That turns the interview into confirmation and correction, which people are
far better at than invention.

Be straight about the limit. You can read what the scoring produced — the tier on each person, and
how many sit in each tier. You cannot see or change how the scores are calculated; that is Athena's
scoring builder, a different surface. If the user wants the weighting changed, say so and route them
there rather than implying you can adjust it.

## Step 4 — The interview

Cover these, in whatever order the conversation wants them:

- **Accounts** — the pharma companies they sell to. Ground them against the `account_names` facet.
- **Role types** and **seniority** — who they sell to inside those accounts.
- **Intent signals** — the triggers worth acting on. The facet is `athena_designations`; the word to
  say out loud is Intent Signals.
- **Geography** — geographical remit, and country or state where it matters.
- **Therapy areas** and **disease areas**.
- **Brands** where the client works brand-by-brand.

Then a set of things that are **not portal filters** but belong in the company context as prose,
because they steer drafting and the intelligence side rather than a search: drug lifecycle stage,
route of administration, company tier and sales tier. Write them down as sentences. Do not invent
filter fields for them and do not promise a search that uses them.

While you go, check each answer with `athena_filter_draft`. A term that comes back in `unresolved`
was IGNORED — say so and offer the suggestions rather than writing a context term that will never
match anything. A term in `ambiguous` needs one question answered before it means anything.

## Step 5 — Propose with counts, then save

Never save a document the user has not seen. Show them:

- the company context you intend to write, in full;
- the number of people it describes, from `athena_filter_draft`, with any caveat the draft reported
  before the number rather than after it;
- for each person you are seeding, the user context you intend to write for them and its count.

Then save with `athena_asset_set`. Three things to know about saving:

- **It goes live immediately.** There is no draft and no review step. That is why the confirmation
  above is the control.
- **Pass `row_version` exactly as `athena_asset_get` returned it.** If someone changed the document
  while you were talking, you get a conflict, nothing is overwritten, and the response carries the
  current live document. Merge your change into that and call again with ITS `row_version`. Never
  retry a conflict by sending the same content again.
- **Check `can_edit` before you offer.** If it is false, the user cannot publish this document —
  `editable_by` names who can. Say that instead of proposing an edit the server will refuse.

Documents are capped at 64 KiB of text. Over that the save is refused with the exact numbers and
nothing is truncated — cut it down rather than hoping.

## Step 6 — Seed each person, and let them confirm

Seeded user contexts are live as soon as they are written. Nobody has to accept them for the system
to work — but the first time each person opens a conversation, their assistant should read their
context back to them and offer to tune it. Say so when you seed: "each person's context is live now,
and they can change their own whenever they want."

Record each person's **function or title** in their user context (e.g. "VP, Medical Affairs"), not just
the accounts and areas they cover. The drafting skill fills the sender line of an outreach email from it
("I lead [function] at …"); a context that omits it leaves every draft with an unfilled placeholder.

People can always edit their OWN context and their own status note. They cannot read each other's —
that is by design, not a permission that can be granted, so do not offer it.

## Step 7 — The playbook

The playbook is what outreach gets drafted from, and it is paired with **Athena's email writing
guide** on the Intelligence Hub connector. If the Hub is connected, call `get_email_writing_guide`
before seeding the playbook and let it steer the interview; if it is not, build the playbook anyway
and say the conditional blocks are worth revisiting with the guide to hand.

The guide's rule that matters MOST here: intent signals come in two tiers, and **a Tier 2 signal is
off-limits to every draft until the playbook carries a conditional block for it**. So do not ask for
blocks in the abstract — read the client's own `athena_designations` facet, tell them which Tier 2
signals actually appear across their contacts, and ask for blocks for the common ones first. Every
Tier 2 signal they skip is a hook their drafts will silently never use; say that plainly, and record
which signals were deliberately left uncovered so the choice is visible rather than an accident.
(Read the tier assignments from the guide itself each time — they are Athena's to change, not this
skill's to remember.)

Seed the playbook with the structure Athena's template uses, and fill what the conversation gives
you:

1. **Company overview and baseline messaging** — what the company does, its differentiators, and two
   or three baseline emails they are happy with.
2. **Conditional messaging by data point** — a block per therapy area, disease area, role type, intent
   signal or brand where they want the framing to change.
3. **Job change and conference guidance** — what to say to someone who has just moved, been promoted,
   or is speaking somewhere.
4. **Company-specific guidance** — one block per account where the framing should change: an existing
   relationship, a prior pilot, something to avoid mentioning.

Fill it from what they tell you, and from their own material where they want to point you at it — case
study decks, past proposals, their website. Read those through **their** tools: their drive, their
document store, or a file they attach. Athena never stores their source documents; only the playbook
you write lands here. If you cannot reach something the playbook needs, ask for it. Do not write a
case study from memory and do not invent a metric.

A playbook with two honest blocks beats one with twelve invented ones. Say that if they stall.

## Step 8 — Offer the recurring briefing

The point of set-up is that the briefing then arrives without anyone asking for it.

Where the platform supports scheduled runs, offer to create one: "shall I set up a monthly run that
does the What's next? briefing and shows you the result?" Monthly, timed for just after Athena
publishes its curated editions, is the right default — that is the cadence the underlying data moves
at, and a weekly run mostly reports that nothing has happened. Conference work moves faster and is
better run on its own, when a conference is actually coming up.

Where the platform does not support it, say so plainly and leave it: an assistant that promises a
recurring run it cannot create is worse than one that says "you will need to ask me each month."

Two things not to rely on:

- **Do not lean on the platform's own approval prompts to protect the writes.** Whether a scheduled
  run is asked to confirm an action varies by platform and by surface, and it is not dependable. The
  protection is that the briefing's only write is idempotent and it advances the baseline only after
  the briefing has actually reached the user.
- **Do not assume an unattended run has your tools loaded.** Write the instruction to load the Athena
  tools into the scheduled task's own prompt.

## Step 9 — Confirm what is live

Close by saying, in plain terms, what now exists: the company context and its version, whose contexts
were seeded, whether the playbook is started or finished, and whether a recurring run was created.
Every document keeps its full history with an author and a timestamp against each version, and
anything can be rolled back from the Athena portal — worth saying once, because it is what makes
direct publishing safe.

One exception, and it needs saying: **the status note keeps no history.** Anything cleared from it is
gone. If you are ever about to clear one, say "this cannot be undone" and get an explicit yes first.
