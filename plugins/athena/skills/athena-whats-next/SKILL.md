---
name: athena-whats-next
description: Produce the Radar Briefing from Athena — what has changed since the user last looked, as a short pointer line and a few numbered sections of things worth acting on, with drill-down. Use when someone asks what's new, what's changed, what they should do next, who has moved, says "what's next on my radar", or when a recurring briefing run fires.
named-terms:
  briefing: Radar Briefing
  pickup-phrase: What's next on my radar
  note: Placeholders. Written here once so a rename is one edit.
---

# Radar Briefing

A briefing, not a data dump. A handful of numbered items the user could act on this week, each one
traceable back to something that actually changed since the last time they were told.

The engine is `athena_changes`. This skill is what turns its output into something a person reads in
ninety seconds, and — just as important — what keeps the "since last time" honest across runs.

## Before you start

**Check you are live by CALLING a tool, not by looking for one.** On some platforms connector tools
are listed but not loaded, so "I can see the tools" proves nothing either way. Call `athena_orient`.
If it returns, you are connected. If it does not, say so and stop — never produce a briefing from
memory or from an earlier conversation.

**If they asked you to PICK something UP, the pick-up gate runs before you plan anything.** "What's
next on my radar", "pick up my Radar Briefing", "where were we" — any of those, and the first thing
you do after `athena_orient` is the gate in "Picking up where they left off" below. It decides
whether a briefing may be produced at all. If the status note holds no markers from before a
previous briefing, the gate FAILS, and a failed gate means you say one sentence and stop — no
`athena_changes` call, no briefing, no caveat. Do not start composing a briefing and then look for a
reason not to send it; the gate comes first, and it is the whole of the answer when it fails.

`athena_orient` carries the vocabulary and the safety rules for this connector. Read them. Two are
worth repeating here because a briefing is exactly where they get broken:

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

**Use the user's other tools.** If their CRM is connected, checking whether someone is already an open
opportunity makes the briefing better. Athena is not trying to be the only thing in the room.

## Step 1 — Load the standing documents

Call `athena_asset_get` three times, at the start of every session:

- `kind: company_context` — who this company sells to.
- `kind: user_context` — this person's patch. Defaults to the caller; that is the one you want.
- `kind: status` — the assistant's own working notes for this person, including the baseline.

`exists: false` is a normal answer, not an error — it means nobody has written that document yet. If
the user context does not exist, say so and offer to set it up rather than briefing them on
everything the company can see, which is what an empty scope produces.

## Step 2 — Read the baseline out of the status note

The baseline is a set of **cursors** — one per stream — and it lives in the status note, because
nothing on the server remembers where a person got to.

Keep it in a fenced block with a marker so it survives alongside whatever else the note holds:

````
## Briefing baseline
<!-- athena:cursors -->
```json
[
  {
    "stream_key": "new_arrivals",
    "last_edition_id": null,
    "last_edition_order_value": "2026-08-01T09:14:00Z",
    "label": "briefing of 1 August"
  },
  {
    "stream_key": "job_change_updates",
    "last_edition_id": "…",
    "last_edition_order_value": "2026-08-04T00:00:00Z",
    "label": "August edition"
  }
]
```
````

Read that array and pass it as the `cursors` parameter. If there is no block, omit `cursors`
entirely — the engine runs a first-run default and says so in every group. Do not invent a date to
stand in for a missing baseline: a guessed baseline either buries the user in history or silently
hides things.

**Never rename or invent a `stream_key`.** They are permanent. `new_arrivals`, `role_changes` and
`employer_moves` are fixed; the series keys come from what the tool returns. A key that does not
match reads as a first run, and the user is told everything is new.

## Step 3 — Build the scope

Turn the user context into a contact filter and pass it as `scope` — but only the parts of it that
are meant to narrow. The context is written in three labelled parts and they do different jobs:

- **Scope** — accounts, role types, seniority. **All three go into the filter**, and accounts go in
  as `account_names`.
- **Exclusions** — only values the user named as unwanted. These go into the filter too.
- **Priorities** — therapy areas, disease areas, geographical remits, intent signals, brands. These
  do **not** go into the filter. They decide what you lead with and what the pointer line says.

**Accounts are Scope, whatever the context calls them.** A context that lists them under "Priority
accounts", "my accounts", or just as a list of company names beside the person's name is naming the
accounts they sell into, and accounts narrow. Reading them as a priority and leaving them out is how
a rep asking about their six accounts gets the whole company's briefing instead — a wrong answer that
looks exactly like a right one, because everything in it is true.

Putting a genuine priority into the filter is the opposite failure: it is how a briefing silently
stops reporting the people it was supposed to rank lower — it drops them instead. The briefing is
deliberately broader than the things the user said they care most about. Use the field names the
filter tools use, grounded against `athena_filter_options_get` if you are unsure a term exists here.

Then read `scope` on the way back out, before you say anything about the report:

- `scope.unresolved` — these terms matched no live value and were **IGNORED**. The briefing therefore
  answers a BROADER question than the user's context describes. Name them: "I couldn't match 'Pfizer'
  in your context — worth checking the spelling, and this briefing covers more than your accounts
  because of it."
- `scope.ambiguous` — not applied at all. Ask which was meant.
- `scope.stripped_fields` — removed because they are not the caller's to set.

An unmatched context term that goes unmentioned is how a confident, empty, wrong briefing gets
produced. Say it first, before the items.

**Only say nothing was ignored when nothing was.** "Your scope resolved cleanly, so nothing in your
context was ignored" is a claim about two things, not one: that `unresolved` and `ambiguous` came
back empty, AND that every Scope and Exclusions part of the context actually went into the filter you
sent. Read the filter you built back against the context before you say it. If a Scope part is
missing from the filter, that part was ignored — by you, silently, which is worse than the server
ignoring it, because the server at least reports it. Do not leave one out; and if you have, say which
one rather than saying nothing was ignored.

## Step 4 — Read each group's state before you describe it

An empty group means four different things and only one of them is good news. Read `state`:

- **`has_items`** — the ordinary case.
- **`nothing_new`** — the stream ran and genuinely found nothing. Say so in one line.
- **`no_newer_edition`** — no new edition of that series has been published. Say which edition is
  still the latest, naming it as the group names it — the `title`, and the edition named in the
  group's `summary`, rather than a series name you are carrying from somewhere else. If the user was
  expecting one, add that the published name may have drifted and it is worth flagging to Athena.
  **Never substitute an older edition and present it as new.**
- **`not_computed`** — no longer produced for the three change streams; the server examines a patch
  of any size now. If you still see it, you are talking to an older server: treat it as "unexamined,
  not empty", offer to narrow the scope, and say the connector is behind. Never report it as nothing;
  nothing and unexamined are not the same answer.

Groups carry a `note` when there is something you need in order to read them correctly. Pass it on
rather than dropping it.

## Step 5 — Render it: one pointer line, then the sections

Open with a single pointer line that says what is in front of them, how much of it there is, and
**which items to start with — named, in the pointer line itself**: "Here is your Radar Briefing: 11
things since 1 August, and the two to start with are Omar Mensah's move to Zenas and the ESMO speaker
list." **Name at most three items, and NAME them.** "The first three are worth your morning" without
saying which three is not a pointer line, and neither is one whose three are revealed at the foot
under a closing "where I would start" — a reader who stops after the first screen has to leave with
the pointer. The recommendation goes at the top, not in a trailer. If the whole briefing is three
items or fewer, skip the pointer line and go straight to them. Then the sections, in this order,
every time. A section with nothing in it still appears, as one line saying so and what was checked;
an absent section reads as a system that forgot rather than a patch that was quiet.

1. **Connections who changed role or employer.** First, always. These are people someone at the
   client already knows, and they are the highest-value flags the briefing makes. Read the
   `connection` marker off the item rather than guessing from a name. If the company holds no
   connection data at all, say that — "Athena holds no LinkedIn connections for your company yet" —
   rather than a line that reads like a quiet month.
2. **Conferences in the next 90 days.** Speakers in their patch, and where Athena has published a
   Likely Attendee cut for the event, the people on it who are already connections — "ESMO is next
   month and five of your connections are likely to be there."
3. **The first curated edition group**, headed by that group's own `title`: the people in this
   user's patch that the edition carries.
4. **The second curated edition group**, the same way. The curated groups keep the order the tool
   returns them in, and they hold positions 3 and 4 whatever they turn out to be called.
5. **Role and employer changes.** Changes since the last briefing that no edition has carried yet.
   Say where an employer move came FROM; that is often the most useful half of the flag.
6. **New arrivals.** The lowest-value flag, and last for that reason.

**Four headings are fixed text; two come off the wire.** Print "Connections who changed role or
employer", "Conferences in the next 90 days", "Role and employer changes" and "New arrivals" exactly
as they stand above. Head sections 3 and 4 with each curated group's own `title`, copied character
for character out of the response.

**Never carry a series name in your own words.** The series, and the editions they publish, are
server configuration rather than something you know: a phrase you remember is a phrase that can be
renamed underneath you, and a section headed by a name the data has stopped using is one the user
cannot match to anything. Read those two headings off the groups every time, even when you are sure
you know them.

However a heading was arrived at, it does not move afterwards. Do not append a qualifier ("… matched
to your patch"), do not re-word one for the second telling, and give a pick-up the same six headings
as the briefing it reproduces. A pick-up whose sections are named differently cannot be checked
against the briefing it claims to be reproducing, which is the only thing the user has to go on.

**Number the headline items. Always.** Number straight through the sections, 1 upward, so "tell me
more about 2" resolves to one thing — there is no way for the user to click one. Keep that numbering
for the rest of the conversation. Each item carries an `item_id` if you need to be exact about which
one you mean.

For each item:

- what changed, in one line, in the user's own vocabulary;
- who it is about and where they work;
- their `lead_score_standardized`, with their `lead_score_tier` name beside it, ordering within a
  section by score;
- why it is worth their time this week.

**Say the date the data supports, and no more.** Each kind of change carries its own date field, and
they are not equally precise:

- **Role changes are month-granular.** `role_start_date` arrives carrying a day, and that day is an
  artefact of the storage: role start dates are RECORDED to the month, and the comparison that
  selected the change ran at month boundaries. Say the month and nothing finer — "now Chief Medical
  Officer at Zenas, started in September". Never "on 8 September", never "from 23 September". A day
  the source cannot support is a fact the user may act on, and it is one the data never had.
- **Employer moves carry a real date, so give it.** `changed_at` is when the move was recorded.
  Render a move as **"moved from X to Y, now <title>, on <date>"** — "Rohan Reyes moved from Intellia
  to 89bio, now Director, Patient Marketing (HAE), on 14 September". The from, the to, the title and
  the date, every time. Dropping the date leaves the user unable to tell last week's move from last
  quarter's, and the field is on the wire whether or not you print it.
- **New arrivals carry `arrived_at`**, the day the contact appeared. A day is right here.

**Print at most five items in a section, but keep the rest.** The tool returns up to ten per group.
When you print five, hold the others in this conversation and say "five more, say the word". "N more"
is counted from what the group says is available, minus what you have printed; never characterise
people you have not been given.

**Keep track of what you have PRINTED, and never print it twice.** Two lists, both held in this
conversation and neither written anywhere: the items you have already printed, and the items the tool
returned that you have not. When they ask for more — "say the word", "the next ten by score", or
anything of that shape — serve the RETURNED-BUT-UNPRINTED queue first, in score order, and call the
continuation only once that queue is empty. Otherwise people six to ten are skipped, because the
continuation starts after the ten the tool has already given you.

Number on from the last number you used: item 11 follows item 10, and the numbering keeps climbing
for the rest of the conversation. An item you printed earlier in this conversation is never one of
the next ones — not renumbered, and not with a note saying it is a repeat. A repeat is not "the next
ten"; it is a shorter answer than the one they asked for. If the queue and the continuation together
run out before you reach the number they asked for, print what there is and say that is all of it.

Where a group reports `result_truncated`, there is more behind it — say how much, and use
`continue_group_id` and `continue_offset` from that group's continuation to fetch the rest when
asked, rather than re-running the whole report.

For a curated edition group, both numbers are worth saying: how many of the edition's people are in
this user's patch, and how big the edition was. "Three of the 180 people in the August list are
yours" reads correctly; a bare "three" makes the edition sound tiny.

**Read the date before you offer the conference.** `list_conferences` and `get_conference` carry
`start_date`; `list_speaker_conferences` carries `start_date` and `year`. `search_speakers` carries
the speaker and no date at all — so a speaker found that way is never offered as a chance to meet
someone until you have opened the conference record and seen a future date. No future date, no offer,
and a past speaking slot is history, not an opportunity.

**The Likely Attendee cut is a prospect list, not a registration list.** Where a conference carries a
`likely_attendees_portal_url`, it resolves to people whose disease areas match the conference's focus,
in the countries the Hub lists. Pass its values to `athena_contact_find` exactly as they appear in the
URL — never through `athena_filter_draft`, which will helpfully widen a term and silently change the
list — and cross it with the user's connections rather than pulling the whole cut, which is both the
useful answer and the only cheap one. **It is often null**, and that is normal: say no Likely Attendee
list is published for that conference rather than inferring one. Whole-list sizes come from the Hub's
own `exact_match_count` and `potential_match_count`, never from a portal count.

### When there is nothing

Say it in one line, name what was checked and from when, and offer the standing alternatives — asking
the data directly, or looking at an upcoming conference. An empty Radar Briefing is still a Radar
Briefing; say which it is so the next one is recognisable. Do not pad an empty result into the shape of
a briefing. A user who learns that "no news" is honest will keep reading the ones that are not.

### Two sections that do not exist yet

If the user asks for their favourites, or for job-change trends over time, tell them plainly that
neither is built yet and that both are planned sections of this briefing. Do not approximate either
from what is available — a hand-rolled trend from a single month's data is worse than the honest
answer.

## Step 6 — Save the baseline, and only after the user has it

`next_cursors` in the response is what the baseline SHOULD become. Three rules govern writing it back,
and all three exist because getting this wrong loses a briefing nobody ever saw:

1. **Advance only on confirmed delivery.** Write the cursors after the briefing has actually reached
   the user — not when the report was generated. A run that fails, errors, or is cut off before the
   user sees anything must leave the baseline exactly where it was.
2. **Merge stream by stream.** Update only the streams present in `next_cursors`, leaving every other
   entry in the note untouched. A scheduled run and a conversation happening the same morning must not
   overwrite each other's progress, and a wholesale replacement is how one of them does.
3. **Retrying is safe; re-reporting is not a failure.** If you are unsure whether the last run's
   cursors were saved, run again with the baseline you have. The same edition reported twice is a
   small annoyance. An edition marked seen and never shown is gone.

Write it back with `athena_asset_set`, `kind: status`, passing the `row_version` from the read at step
1 exactly as it came. On a conflict, nothing was overwritten and the response carries the current
document — merge your cursor block into THAT and call again with its `row_version`. Never resend the
same content after a conflict.

**The status note keeps no version history.** Anything you remove from it is gone. Preserve whatever
else the note holds when you write the cursors back, and never clear the note as part of a briefing.

If `next_cursors` came back with `unrecognised_cursor_keys`, an old key is sitting in the note. Drop
those entries when you write back and mention it once.

**Keep the markers you just moved off.** Alongside the new cursors, record the `baseline_used` values
each group reported for this briefing, and when you delivered it. Those are the markers as they stood
BEFORE this briefing, and they are what a pick-up replays from. Save what `baseline_used` says, not
the clock: the clock slides, and a replay from a sliding window is a different briefing.

**The note holds markers and nothing else.** Not items, not contact ids, not summaries of what you
said, and no record of what the user acted on. It is a set of cursors and two timestamps. Anything
more is a small database inside a text file, and it will drift from the truth the moment the data
moves.

## Picking up where they left off

"What's next on my radar" — or anything that means "let's pick up on my Radar Briefing" — is NOT a
request for a new briefing. Work steps P1 and P2 below IN ORDER. P1 is a gate, not advice: until it
passes you may not call `athena_changes`, and you may not write a briefing.

### Step P1 — The gate: is there a previous briefing to pick up?

Do these five things, in this order, before anything else:

1. **Read the status note first.** Call `athena_asset_get`, `kind: status`, and read it before you
   plan a briefing, build a scope or call `athena_changes`. A pick-up starts by finding out whether
   there is anything to pick up.
2. **Look for the markers from before the last briefing.** That is the block step 6 writes when a
   briefing is delivered: the `baseline_used` values each group reported, with the date you delivered
   it. The current cursor block (`<!-- athena:cursors -->`) is a DIFFERENT thing and finding it is
   not a pass — current markers on their own mean a briefing was delivered by something that did not
   record what it replayed from, so there is still nothing to replay.
3. **If that block is absent, the gate has FAILED. Say exactly this, and nothing else:** "I have no
   previous Radar Briefing to pick up. I can run a fresh one now - it will look back 30 days. Shall
   I?" Then END THE TURN and wait for their answer.
4. **A failed gate is the whole of your answer.** No `athena_changes` call, no scope, no pointer
   line, no sections, no items, and no caveat around a briefing you delivered anyway. Delivering one
   and explaining the gap afterwards is the failure this gate exists to stop: they asked to pick
   something up, so a briefing that arrives reads as the one they were picking up, and nothing you
   write around it tells them otherwise. Offering and stopping is not an unhelpful answer — it is
   the correct one.
5. **Only if that block is present** do you carry on to step P2. If they answer your offer with a
   yes, that is a NEW briefing: go to step 1 at the top of this skill and run it as a first briefing,
   not as a pick-up — no pick-up opening line, and the baseline advances on delivery as normal.

**If there are no earlier markers, offer a briefing — do not deliver one.** Do not infer the
first-run window and produce a full briefing unasked, however well you explain it afterwards.

### Step P2 — Rebuild the last briefing and work it with them

- **Open with this line, in these words.** The first thing they see is, verbatim, carrying the date
  of the briefing you are picking up: "Picking up your Radar Briefing from 16 September - I have not
  moved your marker, so nothing here counts as read". Then the invitation, also verbatim: "tell me
  what is done and I will leave it out for the rest of this conversation". Conveying the same two
  facts in your own words is not enough. "Nothing here counts as read" is the half that explains why
  the same items are in front of them again, and "marker" is the one user-facing word for the
  baseline. The date is the only part that changes.
- **Do not advance the baseline.** Nothing new is being reported, so nothing new has been delivered.
  The cursors stay exactly where they are, and say so: the marker has not moved.
- **Same sections, same headings, same numbers.** Head the sections as step 5 says — the four fixed
  phrases, and the curated groups' own `title`s off this re-run — and give the items the numbers the
  briefing gave them. Renaming a section, or renumbering, makes a pick-up impossible to check against
  the briefing it is reproducing.
- **Rebuild it, do not recall it.** Call `athena_changes` again with the markers you saved before the
  last briefing and the same scope. The engine is stateless, so the same inputs give the same
  briefing and the same item numbers. Present it as a refresh rather than a guaranteed replay:
  anything published or changed since will show, and a one-off cut you made in conversation cannot be
  recovered — say so if they ask for one.
- Work down the items they have not dealt with, and offer the next tranche by standardised score when
  they ask for more, using the continuation the re-run returns — the same printed / unprinted
  discipline as step 5, and the same numbering carried on.
- **Be honest about the limit.** Athena cannot know what they acted on. Unless the outreach happened
  through this assistant, nothing recorded it. Ask rather than assume, and never present a guess as
  memory — it is an invitation to tell you, not an apology.

## When this runs unattended

A scheduled run is the same skill with three differences:

- **Load the Athena tools first.** Put that instruction in the scheduled task's own prompt; a fresh
  unattended run does not inherit this conversation.
- **Delivery is what counts.** The cursors advance only if the briefing actually lands in front of
  the user. If the run cannot deliver, it must not save.
- **Do not depend on the platform asking permission for the write.** Whether an unattended run is
  prompted before it saves varies, and it is not something to rely on. The status-note write is safe
  to repeat, which is the actual protection.

If a run finds nothing at all, say so briefly rather than staying silent — a briefing that goes quiet
is indistinguishable from one that has broken.
