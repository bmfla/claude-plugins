---
name: pitch-a-brand
description: Pitch a brand directly on a creator's behalf — collect the brand name + campaign type + pitch message, show it back for approval, then enqueue the pitch via Creatorland's matchmaker (free to enqueue). Use when the user says "pitch a brand", "reach out to [brand] for me", "send a pitch to [brand]", or "get me in front of [brand]". The creator→brand mirror of cast-and-connect. Enqueuing is free; 10 credits charge ONLY when an approved pitch actually sends. Contact is resolved server-side, the acknowledgement is oracle-safe, the first pitch is human-reviewed — and outcomes are the brand's, never a promised yes.
---

# Pitch-a-Brand

The creator→brand outreach skill — the mirror of `cast-and-connect`, pointed the
other way. A creator names a brand they want to work with, drafts a short pitch,
and Creatorland's matchmaker reaches that brand on their behalf. The creator
NAMES the brand as free text; Creatorland resolves the brand contact
**server-side** — the creator never sees, picks, or supplies a contact. The
deliverable is the enqueued pitch plus a tracking pointer into
`connection-pipeline-tracker`.

Read first: ${CLAUDE_PLUGIN_ROOT}/shared/conventions.md (tool schemas, credit
prices, the conventions) and ${CLAUDE_PLUGIN_ROOT}/shared/connection-flow.md (THE
connection contract — the tool schemas for BOTH directions, entitlement +
graceful degradation, oracle-safe pitch acknowledgement, privacy invariants).

> This is a **connection-enabled** skill: it calls `request_brand_connection`.
> If the user only wants to know which brand categories are active for their
> profile (not to pitch a named brand), use `creator-brand-matchmaker` instead —
> that produces a pitch-target memo and reaches no one.

## Inputs to collect (ask ONLY for what is missing)

- **Brand** (required) — the brand the creator wants to pitch, as **free text**
  (e.g. "Glossier", "On Running"). The creator names it; the contact is resolved
  server-side. Never ask the user for a contact, email, or handle for the brand —
  they don't supply one, and enqueuing never tells you whether the brand is
  reachable.
- **`campaign_type`** (required) — what the creator is proposing (e.g. "paid IG
  Reel + Story", "UGC licensing", "event appearance", "ambassador program").
  Ask for it if not stated.
- **`message`** (required) — the creator's pitch message: who they are, why this
  brand, what they're proposing. Keep it **creator-authored** — never invent
  claims, audience numbers, or offers the creator didn't state.

Never ask twice for anything the user already gave — quote it back instead.

## Flow

**Step A — Assemble + preview the pitch.** Draft the pitch object from the
inputs and **show it back to the user** — brand, `campaign_type`, and the full
`message` — before anything is enqueued. Iterate with the user until they
approve. This preview has zero side effects; nothing is queued or charged until
they say go. Never enqueue without explicit approval of the message.

**Step B — Entitlement probe + graceful degradation.** Do one cheap, free
`list_connections` `{ "limit": 1 }` probe. If the connection layer returns a
**refused / not-entitled envelope** (`brand_connections` not on the plan), do
NOT error: deliver the **drafted pitch** the user approved plus one line —
_"Direct brand pitches aren't on your plan — here's your pitch, ready to send;
you can pitch brands by upgrading to Creatorland Pro (brand connections)."_ Stop
here; the run still succeeded. Only continue when the connection layer is live.

**Step C — Enqueue the pitch (FREE).** On approval, call
`request_brand_connection` once:
```json
request_brand_connection {
  "brand": "<the brand name the user gave, free text>",
  "pitch": {
    "campaign_type": "<required, from inputs>",
    "message": "<the creator-authored pitch, approved in Step A>"
  }
}
```
**Enqueuing is free.** Capture the returned `conn_ref` + status. The
acknowledgement is **oracle-safe** — it is identical whether or not a contact
exists for the named brand, so present it as "your pitch is queued," NEVER as
"we found a contact at <brand>" or any hint about whether the brand is on the
platform.

**Step D — Set expectations (say this every time).**
- The **10-credit charge lands only when an approved pitch actually sends** —
  not on enqueue, and not if the pitch is held or never matched. So this run,
  as enqueued, cost **0 credits**.
- A creator's **first pitch is human-reviewed** before it sends (usually cleared
  same-day); later pitches send without the hold. Frame the review as a feature.
- Outcomes are the **brand's**. A queued pitch is a pitch sent, never a promised
  yes.

**Step E — Write the deliverable** (below) and point the user at the tracker.

## Deliverable

Markdown — the enqueued pitch plus how to track it:

```markdown
# Brand Pitch — <brand>
_Pitched via Creatorland's matchmaker, <date> · Creatorland Data_

## Pitch (queued)
- **Brand:** <brand> (contact resolved by Creatorland — not shown, and not
  something this tool reveals)
- **Campaign type:** <campaign_type>
- **Message:** <the approved, creator-authored pitch>

| conn_ref | Status |
|---|---|
| <conn_ref> | <queued / pending_first_pitch_review> |

Your pitch is **queued** — this acknowledgement does not tell you (and this tool
never reveals) whether a contact exists for <brand>. Creatorland reaches the
brand on your behalf; the outcome is the brand's, never a promised yes.

## What it cost
- Enqueuing was **free**. The **10-credit** charge (~$0.25) applies **only when
  an approved pitch actually sends**, not on enqueue.
- If this is your first pitch, it's **human-reviewed** before sending (usually
  same-day); later pitches skip the hold.

## Next: track it
- Watch the pitch (status, whether it sent) with **connection-pipeline-tracker**
  — it reads `get_connection_status` (by `conn_ref`) / `list_connections` (free).
- A `queued` status is a pitch in flight, **not** a deal.
```

If the entitlement probe (Step B) returned not-entitled, the deliverable is the
**Pitch (queued)** block replaced by the drafted-but-unsent pitch plus the single
upgrade line, and a credit tally of 0 (nothing was enqueued).

## Honesty rules

- **Oracle-safe — never reveal reachability.** The enqueue acknowledgement is
  byte-identical whether or not a contact exists for the named brand. Never imply
  a contact was found, that the brand is (or isn't) on the platform, or that a
  queued pitch means the brand will see it. "Queued," full stop.
- **Contact is resolved server-side.** The creator names the brand; they never
  see, pick, or supply the contact, and no contact info appears in any
  deliverable (conventions 7).
- **Free to enqueue; 10 credits only on an approved send.** Always say this
  before enqueuing and report the actual state (enqueued = 0) after. Never bill
  the enqueue.
- **First pitch is human-reviewed.** Tell first-time pitchers up front; frame it
  as the reason nothing goes out under the Creatorland banner unreviewed.
- **`message` is creator-authored only.** Never invent audience stats, claims,
  or offer terms the creator didn't state.
- **Never promise a yes.** Creatorland runs the outreach; the outcome is the
  brand's. A queued or sent pitch is contact attempted, not a booking.

## Credit footprint

Enqueuing a pitch is **free** (`request_brand_connection` on enqueue;
`list_connections` probe is free). The **10-credit** charge (~$0.25) applies
**only when an approved pitch actually sends** — reported per send, never at
enqueue. Not-entitled plans (`brand_connections` off) incur nothing: the run
degrades to the drafted pitch + an upgrade line. Plan-level facts (grants,
prices) live in ${CLAUDE_PLUGIN_ROOT}/shared/conventions.md.
