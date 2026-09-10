---
name: athena-whats-next
description: Produce the "What's next?" briefing from Athena — what has changed since the user last looked, as a short numbered list of things worth acting on, with drill-down. Use when someone asks what's new, what's changed, what they should do next, who has moved, or when a recurring briefing run fires.
---

# What's next?

A briefing, not a data dump. A handful of numbered items the user could act on this week, each one
traceable back to something that actually changed since the last time they were told.

The engine is `athena_changes`. This skill is what turns its output into something a person reads in
ninety seconds, and — just as important — what keeps the "since last time" honest across runs.

## Before you start

**Check you are live by CALLING a tool, not by looking for one.** On some platforms connector tools
are listed but not loaded, so "I can see the tools" proves nothing either way. Call `athena_orient`.
If it returns, you are connected. If it does not, say so and stop — never produce a briefing from
memory or from an earlier conversation.

`athena_orient` carries the vocabulary and the safety rules for this connector. Read them. Two are
worth repeating here because a briefing is exactly where they get broken:

- **Scores have three states.** `lead_score` of 0 is a real score at the bottom of the ranking, absent
  means never scored, and a `lead_score_tier` of `"N/A"` means the person's role type is deliberately
  not relevant to this client. Never surface an `"N/A"` person as though they were merely unscored.
- **Tier names only.** Pass them through exactly as they arrive. Never convert one to a number, a
  band or a percentile, never rank arithmetically, never recompute one.

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

Turn the user context into a contact filter and pass it as `scope`: their accounts, their role types,
their therapy areas. Use the field names the filter tools use, grounded against
`athena_filter_options_get` if you are unsure a term exists here.

Then read `scope` on the way back out, before you say anything about the report:

- `scope.unresolved` — these terms matched no live value and were **IGNORED**. The briefing therefore
  answers a BROADER question than the user's context describes. Name them: "I couldn't match 'Pfizer'
  in your context — worth checking the spelling, and this briefing covers more than your accounts
  because of it."
- `scope.ambiguous` — not applied at all. Ask which was meant.
- `scope.stripped_fields` — removed because they are not the caller's to set.

An unmatched context term that goes unmentioned is how a confident, empty, wrong briefing gets
produced. Say it first, before the items.

## Step 4 — Read each group's state before you describe it

An empty group means four different things and only one of them is good news. Read `state`:

- **`has_items`** — the ordinary case.
- **`nothing_new`** — the stream ran and genuinely found nothing. Say so in one line.
- **`no_newer_edition`** — no new edition of that series has been published. Say which edition is
  still the latest ("the August Job Change list is still the newest"). If the user was expecting one,
  add that the published name may have drifted and it is worth flagging to Athena. **Never substitute
  an older edition and present it as new.**
- **`not_computed`** — the scope was too broad to examine. Offer to narrow it — accounts, role types,
  therapy area — and run again. Do not report it as nothing; nothing and unexamined are not the same
  answer.

Groups carry a `note` when there is something you need in order to read them correctly. Pass it on
rather than dropping it.

## Step 5 — Render it as a numbered list

**Number the headline items. Always.** There is no way for the user to click one, so "tell me more
about 2" has to resolve to something. Number them in the order you present them and keep that
numbering for the rest of the conversation.

Aim for five to eight items in the headline. For each:

- what changed, in one line, in the user's own vocabulary;
- who it is about and where they work;
- their tier by NAME where it is set, and nothing derived from it;
- why it is worth their time this week.

Then offer the drill-down: "tell me more about any of them." Each item carries an `item_id` if you
need to be exact about which one you mean. Where a group reports `result_truncated`, there is more
behind it — say how much, and use `continue_group_id` and `continue_offset` from that group's
continuation to fetch the rest when asked, rather than re-running the whole report.

For a curated edition group, both numbers are worth saying: how many of the edition's people are in
this user's patch, and how big the edition was. "Three of the 180 people in the August list are
yours" reads correctly; a bare "three" makes the edition sound tiny.

### When there is nothing

Say it in one line, name what was checked and from when, and offer the standing alternatives — asking
the data directly, or looking at an upcoming conference. Do not pad an empty result into the shape of
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
