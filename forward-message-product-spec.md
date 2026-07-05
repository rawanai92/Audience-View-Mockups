# Forward a message — Product Spec (WhatsApp · v1)

**Module:** Conversations · **Channel:** WhatsApp only · **Spike:** [86exwg5f0](https://app.clickup.com/t/90181619228/86exwg5f0)
**Mockup:** `mockup-forward-message.html`

---

## Context

Agents regularly need to pass a message they're looking at into another conversation — a product photo, a price list, a document, a useful answer. Today they re-type it or screenshot it.

WhatsApp's Business API has **no native forward operation**. "Forward" is implemented by re-sending the original message's content as a new message on the existing send pipeline. Two consequences product is signing off on:

1. **The "Forwarded" tag lives in our inbox, not on the customer's phone.** Meta does not let businesses set the grey "Forwarded" label when sending via the API, so the customer receives a normal-looking message. Only our agent inbox shows it as forwarded. This is a hard Meta limitation, not a scoping choice — it needs clear comms to agents.
2. **Because we re-send from our own data, we can be richer internally** — we attach provenance (who it came from, source conversation, original time) and show it to agents.

---

## Scope

### In (v1)
- Single-message forward from the **⋯ menu** on a message bubble.
- Content: **text** and **media** (image / video / audio / document). Media reuses the copy we already store — no quality loss.
- **One destination** per forward, chosen from **any existing conversation** in the same tenant — open *or* closed window. The agent is never blocked from choosing a recipient; the flow adapts to what's actually deliverable (see §"Post-24h").
- A **"truth pill"** in the forward footer that states what will happen *before* send — instant, queued (reopen via one-off template), or not-possible (audio post-24h) — resolved from `window state × content type`.
- In-inbox **"Forwarded"** label + **"Forwarded from …"** provenance line on the destination bubble.

### Out (v1)
- Bulk / multi-select forward.
- Forwarding **interactive / button-reply** messages and reactions (no meaningful re-send shape).
- Forwarding to a brand-new contact (no existing conversation) — Phase 2.
- Jump-to-original from the provenance line — Phase 2.
- "Forwarded many times" depth styling — Phase 3.
- Any "Forwarded" tag on the customer's device (Meta limit) and edit/recall after send (no Meta API).

> Note: forwarding *templates as content* is out, but the **one-off template send path is in** — it's how text and media reach a closed window (below).

---

## The experience

1. Agent hovers a message → **⋯** appears → **Forward**.
2. **"Forward to…"** panel opens with a destination search (same picker pattern agents already use) and a preview of what's being forwarded.
3. (Media) optional caption tweak.
4. Confirm → the message appears in the destination conversation, tagged **"Forwarded"** with **"Forwarded from {Contact / Agent}."**
5. The customer receives the content as a normal message (no forwarded tag — see Context).

---

## Post-24h: what "forward to anyone" actually means

After 24h of customer silence, WhatsApp only delivers **approved templates**. The agent can still pick *any* destination; forwarding into a closed window reopens the chat by sending the content **as a one-off template** (auto-created → Meta approval, usually minutes → delivered → reopens chat → auto-deleted ~24h later). The **truth pill** states the outcome before send:

| Original message | Inside 24h | After 24h (window closed) | Pill |
|---|---|---|---|
| **Text** | Instant, verbatim | One-off **text** template. *(Mechanism exists today.)* | 🕓 *Queued · reopens chat* |
| **Image** | Instant, media re-used | One-off **image-header** template. | 🕓 *Queued · reopens chat* |
| **Document** | Instant, media re-used | One-off **document-header** template. *(New build — see below.)* | 🕓 *Queued · reopens chat* |
| **Video** | Instant, media re-used | One-off **video-header** template. *(New build.)* | 🕓 *Queued · reopens chat* |
| **Audio** | Instant | **Not possible** — WhatsApp has no audio template header. Blocked with a reopen workaround. | 🚫 *Not available for audio* |

**Media post-24h — how it's delivered:** the file rides in the template **header**; a short **agent-editable line of text** becomes the body (default e.g. *"Hi 👋 here's the file you asked about."*). The customer receives the file + that line.

**Two honesty points to message:**
1. The body line **replaces the original caption** — shown in the panel before send ("Customer receives: [your file] + the line above").
2. Post-24h is **not instant** — it waits on Meta approval (usually minutes). The bubble shows a pending state until delivered.

**Build note:** text + image one-offs leverage existing support; **document and video headers are net-new** (today only IMAGE headers are wired, hardcoded). Audio is the only true Meta dead-end.

---

## Acceptance criteria

- **AC1** — Hovering any text or media (image/video/document/audio) message reveals a ⋯ menu containing **Forward**. Interactive/button messages and reactions do **not** expose Forward.
- **AC2** — Forward opens a panel showing (a) a preview of the message and (b) a searchable list of the agent's conversations in the same tenant — **open and closed windows alike**; no destination is hidden or disabled at selection time. Only one destination can be selected.
- **AC3** — For media, the agent can edit the caption before sending; the media itself is re-used (not re-uploaded by the agent).
- **AC4** — A footer **truth pill** updates on destination selection and states the outcome before send: *Sends instantly* / *Queued · reopens chat* / *Not available for {type}*.
- **AC5 (inside 24h)** — Any supported content (text + all media) forwards instantly and the destination bubble renders the **"Forwarded"** label + **"Forwarded from {sender}"** provenance line.
- **AC6 (closed window · text)** — Forwarding text reuses the one-off text template mechanism: queued PENDING → delivered on approval → reopens chat → auto-deleted per retention. Bubble shows pending until delivered.
- **AC7 (closed window · image/document/video)** — Forwarding sends a one-off **media-header** template: the file is the header, an agent-editable line is the body. The panel shows "Customer receives [file] + line" and that it **replaces the caption** before confirm. Queued PENDING → delivered on approval → reopens chat → auto-deleted.
- **AC8 (closed window · audio)** — Forwarding audio is **blocked** with a clear reason + reopen workaround + optional "notify me when they reply"; no message is sent.
- **AC9** — Provenance (original sender, source conversation, original time) is **internal-only** — never sent to Meta or shown to the customer.
- **AC10** — Source and destination must belong to the **same tenant**; cross-tenant and cross-channel forwarding are not offered.

---

## UX / Content

- **Label:** grey/italic **"Forwarded"** on the destination bubble (mirrors WhatsApp's own styling so it reads as familiar).
- **Provenance line:** "Forwarded from {Contact name}" or "Forwarded from {Agent / Bot}". Static text in v1 (not yet a link).
- **Picker:** reuse the existing conversation-search component; show contact name, last-activity time, and a window-state hint where the 24h window is closed.
- **Customer-comms note:** because the customer sees a normal message, agent-facing copy should never promise the recipient will "see it was forwarded."

---

## Phased delivery

- **Phase 1 (this spec — MVP):** ⋯-menu → destination picker → re-send → "Forwarded" label + "Forwarded from" provenance, tenant-scoped, existing open conversations only.
- **Phase 2 — Reach & polish:** forward to a brand-new contact (window-gated), richer picker, clickable jump-to-original.
- **Phase 3 — Parity extras (optional):** forward-depth → "Forwarded many times"; degrade interactive messages to text; template forwarding if needed.

---

## Decisions product is making for v1
*(open questions from the spike — resolved here as defaults; flag if any should change.)*

| Question | v1 decision |
|---|---|
| Destinations | **Any existing conversation, open or closed** — agent picks freely; the flow adapts (new contact → Phase 2). |
| Closed-window behavior | **Text + Image + Document + Video** → one-off template reopen (file in header + editable body line for media). **Audio** → blocked (no Meta header). |
| Who can forward | All agents (no role gate in v1). |
| Provenance visibility | Show full "Forwarded from {original sender}" to every agent. *Revisit if customer-identity privacy is a concern.* |
| Jump-to-original | Static label in v1 (clickable → Phase 2). |
| "Forwarded many times" | Skipped in v1 (→ Phase 3). |

### Open follow-ups for the closed-window path
- **Default body line** per type and whether agents can leave it blank (Meta requires a body for media templates).
- **Approval-rejection UX:** if Meta rejects the one-off (content policy), how is the failed forward surfaced and retried?
- **Template quota:** one-off per forward consumes WABA template quota — confirm the dedup/reuse covers the common cases enough to avoid churn.
- **"Notify me when they reply"** (audio blocked state): real v1 capability or just messaging?
- **Build sequencing:** text + image first (least new work), then document + video headers — ship together or staged?

---

## Definition of done

- Forward is available from the ⋯ menu for text + media messages and unavailable for templates/interactive/reactions.
- Forwarded messages deliver via the existing send pipeline and respect the 24h window exactly like a normal reply.
- Destination bubble shows the "Forwarded" label + provenance; provenance never leaves our system.
- Source↔destination tenant isolation is enforced.
- Agent-facing copy sets the correct expectation: the customer does **not** see a forwarded tag.
