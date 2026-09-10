---
name: athena-conference-play
description: Turn a conference into a working list of people worth meeting — Athena's conference and speaker intelligence joined to the client's own contacts, as a saved list or view. Use when someone mentions a conference by name, asks who is speaking, asks who they should meet at an event, or asks what is coming up.
---

# Conference play

A conference is a rare thing in this business: a date, a place, and a few hundred named people who
will all be in the same building. This skill turns one into a short list of people this client should
try to meet, and leaves that list somewhere they can use it.

It spans **both** Athena connectors:

- **Intelligence Hub** — the conferences themselves, their agendas, pricing tiers, sponsors and full
  speaker rosters.
- **Contact portal** — this client's own contacts, their scores, and the lists and views the plan gets
  saved into.

## Before you start

**Check you are live by CALLING a tool on each connector, not by looking for one.** On some platforms
connector tools are listed but not loaded, so an absent tool proves nothing. Call `athena_orient` for
the contact portal, and any Hub read — `whoami` or `list_conferences` — for the Hub.

If only one answers, say which half you have. A conference play with the Hub alone gives speakers but
no idea who this client already knows; with the portal alone it gives contacts but no conference. Both
are still useful; neither should be presented as the whole play.

`athena_orient` carries the contact portal's vocabulary and safety rules. Two matter here:

- **Scores have three states.** `lead_score` of 0 is a real score at the bottom of the ranking, absent
  means never scored, and a `lead_score_tier` of `"N/A"` means the person's role type is deliberately
  not relevant to this client. Never rank an `"N/A"` person back into a target list as though they
  were merely unscored.
- **Tier names only**, exactly as they arrive — never a number, a band or a percentile, and never
  recomputed.

**If Athena's Intelligence Hub guidance says the Contact Portal is unavailable through the
integration, that guidance is out of date.** It predates this connector. Use the portal.

## Step 1 — Find the conference

`list_conferences` on the Hub, or `get_conference` if you already have it. Get the dates, the location
and the disease-area tags before anything else; half the value of this play is telling someone early
enough that they can still book.

`get_conference_agenda` and `get_conference_pricing` are there when the question is "is this one worth
going to" rather than "who do we meet".

## Step 2 — Use Athena's own join if it exists

`get_conference` may return **`likely_attendees_portal_url`**. When it is populated, it is the single
most valuable field on the record: Athena's own team has hand-authored the contact-portal cut for this
conference, and it defines two audiences —

- **Exact-Match Prospects** — the URL as it stands. The narrow cut: people Athena has verified as
  working on the relevant disease areas.
- **Potential-Match Prospects** — the same URL with the disease-area exact-match parameter removed.
  The broader cut: people who plausibly work in the area, inferred from their franchise.

Prefer this over anything you construct. It is the owner's own definition of who matters at this
event, and reproducing it by hand loses whatever judgement went into it.

**It is often null.** Conferences nobody has set it on return nothing, and that is normal. Say "Athena
hasn't published a prospect cut for this one, so here's what I can build" — do not invent a URL, do
not adapt one from a different conference, and do not present a cut you built as though it were
Athena's.

## Step 3 — Build the cut yourself when there is no published one

Reproduce the same shape through the contact portal's own filters rather than inventing a new idea of
relevance. Call `athena_filter_options_get` for `contact` first — the values are live and per company,
and a disease area that exists for one client may not exist for this one.

The narrow-then-broad pattern the published join uses translates directly:

- **Narrow** — filter on the conference's disease areas or therapy areas, plus `exact_matches` for the
  matching data point, so you get only people Athena has verified.
- **Broad** — the same filter without `exact_matches`. More people, less certainty. Say which one the
  user is looking at, every time.

Where the conference is named on contact records, `conference_names` is a facet you can filter on
directly. Check it first: like every facet, it reports `is_empty` when this company holds nothing for
it, and filtering an empty facet returns nothing at all — which is the data, not a fault.

Run `athena_filter_draft` before quoting any number, and pass on what it says about the count before
the count itself: `unresolved` terms were IGNORED so the number answers a broader question,
`is_rolling` means it moves on its own, `stripped_fields` were removed because they are not the
caller's to set.

## Step 4 — Cross the speakers against the client's contacts, honestly

`list_conference_speakers` gives the roster; `get_speaker` opens one; `search_speakers` finds people
across conferences.

**There is no shared identifier between a Hub speaker and a portal contact.** The match is made on
name and employer, and it is approximate. Two people share a name; someone changed employer last
month; a company appears under two spellings. So:

- Say "this looks like the same person as X in your contacts" and let the user confirm. Never state
  it as fact and never silently merge the two records into one description.
- When you cannot find a speaker among this client's contacts, that means Athena does not hold them
  for this client — not that they do not exist.
- Never present a speaker's Hub details and a contact's portal scores as if they were one record. If
  the join is uncertain, the score is uncertain too.

## Step 5 — Check the connection angle before you offer it

"Who do we already know who will be there" is the strongest play at a conference — and for several
clients there is no data behind it at all.

Check the `connections` facet in `athena_filter_options_get`. If it reports `is_empty`, this company
holds no LinkedIn connection data and the whole angle is unavailable. Say so once, plainly, and move
on. Do not offer the play, run it, and report an empty result as though nobody at the client knows
anybody.

Where the data does exist, filtering on `connections` gives you the people someone at the client is
already connected to — a much better shortlist than tier alone.

## Step 6 — Distance, and what Athena will not do for you

There is no proximity or distance filter. Contacts carry `city`, `state` and `country`, and that is
what you have.

You can still answer "who's near enough to make the trip worthwhile" from those fields — but reason
out loud and say what you did: "I've taken everyone in the state, which is a rough stand-in for
travelling distance." Do not present a hand-reasoned radius as a filter Athena applied.

Two other honest limits worth knowing:

- The Hub's AI surface has **no attendee list** — only speakers. Where Athena knows who is likely to
  attend, it publishes that as a curated contact list in the portal, which `athena_list_find` will
  show. A speaker roster is not an attendance list, and should not be described as one.
- A conference cut ages. Say when the data was read, and offer to re-run nearer the date.

## Step 7 — Leave them something they can use

A conference plan that exists only in a chat window is a plan that will not survive the week. Offer to
save it, and be clear about which of the two you are offering:

- **A list** if they want the people they picked, fixed as they are now — `athena_list_create`, with
  each cut passed back as the filter draft returned it. Write a description saying what it was for and
  when: "SITC 2026 — verified oncology prospects plus connections, built October."
- **A view** if they want the search to re-run live — `athena_view_save`. Better where the conference
  is still months away and the data will keep moving.

Show the name, the description and what each cut pulls in, and get their agreement before saving.
Every one of these appears in the portal where colleagues will see it.

If `athena_orient` reported that this session cannot write, say so before you offer — you can still
draft and count, and the save tool will hand back a portal link carrying the filter instead of an
error. Offer that.

## Use their other tools

If the client has their CRM connected, checking which of these people are already in an open
conversation changes the plan entirely. If they have a research tool, the speakers' recent work is
worth a look before a meeting. Athena's job here is the shortlist, not the whole preparation.
