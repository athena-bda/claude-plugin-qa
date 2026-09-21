---
name: athena-set-up
description: Set up or tune a client on Athena — record who the company sells to, who each person covers, and the messaging playbook they draft from, so every later Athena conversation is already scoped. Use when someone says "set me up", "set up Athena", "tune my context", "help me build my messaging playbook", or when a briefing comes back too broad because nobody has said what the user actually cares about.
named-terms:
  briefing: Radar Briefing
  pickup-phrase: What's next on my radar
  note: Placeholders. Written here once so a rename is one edit.
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

- **Scores have three states, and one number.** `lead_score_standardized` is the only lead score you
  will see or say. `lead_score_standardized` absent means not scored on this client's standardised
  scale. `lead_score_standardized` 0 with a tier is a REAL score, at the bottom of the ranking.
  `lead_score_standardized` 0 with `lead_score_tier` "N/A" means one of this client's own scoring
  rules ruled the person out: a value they marked as unwanted, with nothing scored to outweigh it.
  Say "ruled out by your scoring rules", never "unscored", and keep them out of priority lists. A
  ruled-out person's score is 0 too, so the TIER is the only thing that tells the two apart. N/A on a
  data field — therapy area, remit, and so on — is a different thing entirely: it means Athena has no
  information. Say "unknown", and never exclude anyone on it.
- **Tier names only**, exactly as they arrive. Never turn one into a number, a band or a percentile,
  and never recompute a score. A tier is a LABEL, never a filter of its own: prioritise by
  `lead_score_standardized`, present the tier name beside it, and never make a tier the sole reason
  to include or leave someone out. "Highest priority" means the top of the score ordering, not one
  named tier. When someone asks for a count, walk down the score ordering until you have that many
  and say each one's tier as you go — a High at 51 and a Medium at 50 are neighbours.

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

## Step 3 — Start from the client's own scoring rules

Call `athena_scoring_rules_get` before you ask anything. It returns the scoring rules set up for this
client: which values of which properties earn points, and which values rule a contact out entirely.
Those are decisions the client has already made, and starting from them turns the interview into
confirmation and correction — which people are far better at than invention.

**If it returns rules**, read them and propose. Say what the rules already decide — "the scoring set
up for you ranks Neurology, Oncology and Rare Diseases highest, rules out eleven role types, and
marks Latin America, the Middle East and Africa down; we can change any of it" — and ask the user to
confirm or correct each one. Where what the user says contradicts what the rules do, say so plainly
and ask which is right: that contradiction is one of the most useful things this conversation can
surface.

**A ruled-out value is not an exclusion you may apply on their behalf.** Present it as what it is —
"your scoring marks these as unwanted, so someone carrying only one of them is ruled out; someone who
also carries a scored value is kept" — and still ask topic 4's exclusion question separately. An
exclusion comes from what the user says they do not want, never from a rule row.

**If it returns no rules**, or says it could not read them in full, say so in one line and run the
interview cold, from topic 1. That is a normal state, not a fault. It never means the client has not
been scored.

Once a topic is confirmed, a count is still worth putting in front of them: draft it with
`athena_filter_draft` so the shape of the population is visible before anything is written down.

Two limits to be straight about either way:

- The rules do not usually carry accounts or brands. Those topics are real questions every time.
- You can read what the scoring decided; you cannot change how it is calculated in this conversation.
  If the user wants the weighting changed, use these words: **"these are the scoring rules set up for
  you, which we can change — ask Athena"**. Do not name the place the rules are authored and do not
  send the user to it. It is not a surface a client can reach, so a route to it is a door that does
  not open, and an instruction to go there reads as the user's job left undone. This holds in the
  conversation AND in anything you publish: never write the name of an internal surface into a
  context document, where it outlives the conversation and the whole company reads it.

Do not ask for, infer, or state a tier threshold. Where a tier boundary sits is not something this
conversation needs and not something you are given.

## Step 4 — The interview

Cover all seven, in this order, every time. Ask each one; do not wait for the user to notice a topic
was skipped. Where the scoring rules already answer a topic, put the answer in front of them to
confirm rather than asking cold. If they answer several topics at once, take the lot and confirm each
remaining one in a line rather than making them repeat themselves.

1. **Accounts, and which of them are priorities.** The pharma companies they sell to, grounded
   against the `account_names` facet — and then which of those matter most. The scoring rules do not
   usually carry accounts, so this one is always a real question.
2. **Role types and seniority.** Who they sell to inside those accounts.
3. **Intent signals.** The triggers worth acting on. The facet is `athena_designations`; the words to
   say out loud are Intent Signals.
4. **Geographical remit priorities, and anything to leave out.** Ask which remits are priorities, and
   say more than one can be chosen. Offer the values exactly as the facet returns them — Europe and
   European Region are two different values with two different meanings, and merging them loses the
   distinction. Then ask separately whether there is any remit they definitely do not want. Only that
   second answer becomes an exclusion.
5. **Therapy areas.** Which ones matter most.
6. **Disease areas that are very high priority.** This promotes people; it never narrows the
   briefing. Do not turn it into a filter that leaves anyone out.
7. **Assets or brands** where the client works brand by brand.

Two rules run across all seven:

- **Priorities are not exclusions.** Everything above steers what gets surfaced FIRST. Nothing above
  removes anyone from view unless the user named a specific value they do not want — for example an
  African or a Local affiliate remit. Ask both questions and keep the answers apart.
- **Never exclude on an unknown.** A contact whose remit, therapy area or disease area is N/A is a
  contact Athena has no information about. They stay in.

Write what you learn into the context in three labelled parts, so a later skill can tell them apart:
**Scope** (accounts, role types, seniority), **Priorities** (therapy areas, disease areas, remits,
intent signals, brands) and **Exclusions** (only values the user named as unwanted). Briefings and
views filter on Scope and Exclusions, and rank on Priorities.

**Every context takes those three headings — the company's, and every person's.** Use the three
words themselves as headings, in that order, in every context this interview writes. A context
written as prose, or under headings of your own, gives the skills that read it back nothing to tell
apart, and they are the skills that decide who is in a briefing.

**Accounts go under Scope, even when the user calls them their priority accounts.** Topic 1 asks
which accounts matter most, and the answer to that is a sentence INSIDE Scope — "six accounts, and
Pfizer and Novartis matter most" — never a heading of its own. A heading like "Priority accounts"
makes the next skill read them as a priority and leave them out of the filter, and the briefing that
comes back covers the whole company instead of the accounts the person actually sells into. Accounts
narrow. Always.

Do not ask which tier the briefings should cover. Tiers are a label on what the client's own scoring
produced, not a question for the user.

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

**Say what publishing means, once, before the first save.** In one line, in their words: this goes
live for the whole company straight away, every version is kept with who wrote it and when, and it
can be rolled back from the portal. The status note is the exception — that one is your own working
notes for that person, it overwrites in place and keeps no history. Say it once, at the first save,
not every time.

Then save with `athena_asset_set`. Three things to know about saving:

- **It goes live immediately.** There is no draft and no review step. That is why the confirmation
  above is the control.
- **Pass `row_version` exactly as `athena_asset_get` returned it.** If someone changed the document
  while you were talking, you get a conflict, nothing is overwritten, and the response carries the
  current live document. Merge your change into that and call again with ITS `row_version`. Never
  retry a conflict by sending the same content again.
- **Check `can_edit` before you offer.** If it is false, the user cannot publish this document, and
  `editable_by` names who can — tell them who to ask, in plain words, instead of proposing an edit
  the server will refuse. `editable_by` is filled in ONLY when `can_edit` is false, so an empty one
  beside `can_edit: true` is the normal state and never a fault. Do not narrate it, and never read a
  field name out to the user.

Documents are capped at 64 KiB of text. Over that the save is refused with the exact numbers and
nothing is truncated — cut it down rather than hoping.

## Step 6 — Seed each person, and let them confirm

Seeded user contexts are live as soon as they are written. Nobody has to accept them for the system
to work — but the first time each person opens a conversation, their assistant should read their
context back to them and offer to tune it. Say so when you seed: "each person's context is live now,
and they can change their own whenever they want."

Write each person's context in the same three labelled parts as the company's — **Scope**,
**Priorities**, **Exclusions** — and use step 4's mapping field for field, not just its headings:
**Scope is accounts, role types and seniority; Priorities is therapy areas, disease areas,
geographical remits, intent signals and brands; Exclusions is only the values this person named as
unwanted.** Nothing about seeding changes that mapping. It is easier to break here than in the
company context, because the person you are writing about is usually not in the room to notice that
their territory has been filed in the wrong part.

**A remit under Scope silently narrows everything they will ever see.** The skills that read a user
context FILTER on Scope and Exclusions, and RANK on Priorities. Write "Territory. Remits Europe,
European Region and Global." under Scope and every briefing and every saved view that person gets is
cut down to those three values — no error, no warning, and the global account they cover quietly
stops appearing. A remit is a priority. So are therapy areas, disease areas, intent signals and
brands: each one under Scope removes people instead of ranking them, and a person's context is the
one that decides whose briefing they get. Written as a paragraph, or with their accounts under a
priorities heading, it hands them the whole company's briefing labelled as their own patch.

Record each person's **function or title** in their user context (e.g. "VP, Medical Affairs"), not just
the accounts and areas they cover. The drafting skill fills the sender line of an outreach email from it
("I lead [function] at …"); a context that omits it leaves every draft with an unfilled placeholder.
Put it on a line ABOVE the three headings, as the example does: it says who this person IS, and under
Scope it reads as a seniority they sell to and filters their own briefing down to their own job title.

A seeded user context, in full — accounts a plain list under Scope, remits under Priorities:

```
Function: VP, Medical Affairs.

## Scope
- Pfizer
- Novartis
- AstraZeneca

Pfizer and Novartis matter most. Medical Affairs and Market Access, Director level and above.

## Priorities
Geographical remits: Europe, European Region, Global.
Therapy areas: Oncology, Immunology.
Disease areas: non-small cell lung cancer.
Intent signals: the ones this client's own facet actually returns.
Brands: only where they work brand by brand.

## Exclusions
Local affiliate remits — named by her as unwanted.
```

Europe and European Region are two values, not one, and both are listed under Priorities exactly as
the facet returns them. The Exclusions line is there because she named it, not because a remit was
left out of the priorities.

People can always edit their OWN context and their own status note. They cannot read each other's —
that is by design, not a permission that can be granted, so do not offer it.

## Step 7 — The playbook

**This one is optional, and it is the last thing we do — we can do it now or later.** Say that, and
mean it. A client who wants to stop after their contexts are published has a working set-up.

The playbook is what outreach gets drafted from, and it is paired with **Athena's email writing
guide** on the Intelligence Hub connector. If the Hub is connected, call `get_email_writing_guide`
before seeding the playbook and let it steer the interview; if it is not, build the playbook anyway
and say the conditional messaging is worth revisiting with the guide to hand.

Lead with what it buys them, not with how it is structured: the more approved messaging they supply,
the more every draft can be tailored to the person it is going to; without it drafts stay generic and
say the same thing to everyone. Never say "Tier 1", "Tier 2" or "blocks" to the user — those are the
guide's internal vocabulary and mean nothing to the person answering.

Then make it concrete. Read the client's own `athena_designations` facet, tell them which signals
actually appear across their contacts, and for each of the common ones ask what they would want said
when it is present. Record which signals were deliberately left uncovered, and say plainly that a
signal with nothing written for it is a hook their drafts will silently never use — so the choice is
visible rather than an accident. (Read the guide's own handling of those signals from the guide each
time; it is Athena's to change, not this skill's to remember.)

**Ask for their material, and read it here.** The single most useful thing this step can do is work
from what they already have: invite them to paste or attach approved messaging, case studies and
positioning into the chat, and draft the conditional messaging from those rather than from a blank
page. Show what you took from each and what you left out, so nothing is quietly dropped.

Seed the playbook with the structure Athena's template uses, and fill what the conversation gives
you:

1. **Company overview and baseline messaging** — what the company does, its differentiators, and two
   or three baseline emails they are happy with.
2. **Conditional messaging by data point** — one piece of conditional messaging per therapy area,
   disease area, role type, intent signal or brand where they want the framing to change.
3. **Job change and conference guidance** — what to say to someone who has just moved, been promoted,
   or is speaking somewhere.
4. **Company-specific guidance** — one piece per account where the framing should change: an existing
   relationship, a prior pilot, something to avoid mentioning.

Fill it from what they tell you and from the material they give you here — approved messaging, case
studies, positioning, past proposals, their website. Read those in this conversation, or through
**their** tools: their drive, their document store, or a file they attach. Show what you took from
each and what you left out. Athena never stores their source documents; only the playbook you write
lands here, so do not offer to keep the files. If you cannot reach something the playbook needs, ask
for it. Do not write a case study from memory and do not invent a metric.

A playbook with two honest pieces beats one with twelve invented ones. Say that if they stall.

## Step 8 — Offer the recurring briefing

The point of set-up is that the briefing then arrives without anyone asking for it.

Where the platform supports scheduled runs, offer to create one: "shall I set up a monthly run that
does the Radar Briefing and shows you the result?" Monthly, timed for just after Athena
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
