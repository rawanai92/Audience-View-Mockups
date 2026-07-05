# Forward post-24h — decision brief (Block vs. Create templates)

**For:** Product team approval · **Feature:** Forward a message (WhatsApp) · **Spike:** [86exwg5f0](https://app.clickup.com/t/90181619228/86exwg5f0)

## The constraint
Inside the 24h window, forwarding is easy — we re-send the original as a new message (text or any media; media is already mirrored to our storage, so no re-upload). **After 24h, WhatsApp only delivers approved templates** — free-form text and ad-hoc media are blocked by Meta. So the only open question is: **what happens when an agent forwards into a closed window?**

Two ways to handle it.

---

## Option A — Block (in-window only)

**How it works:** Forwarding is allowed only while the 24h window is open. If it's closed, the forward is blocked with a clear reason and the agent is pointed to the normal reopen path (send a template/text reply first).

| Content type | Inside 24h | Window closed |
|---|---|---|
| Text | ✅ instant | 🚫 blocked + "reopen first" |
| Image / Video / Document / Audio | ✅ instant | 🚫 blocked + "reopen first" |

**Agent experience:** Picks any destination. If closed, sees "Can't forward after 24h — send a message to reopen the chat first," optionally "notify me when they reply." Nothing is faked.

**Effort:** **Low.** No new backend; just the in-window forward + the blocked state.

**Pros:** Simplest, fastest, fully honest, no template clutter, no approval delay. **Cons:** Agent can't forward into a quiet conversation at all without manually reopening it first.

---

## Option B — Create templates (one-off)

**How it works:** When the window is closed, we turn the forwarded content into a template so it can actually be delivered, using the **one-off template mechanism that already exists for text**: auto-create → submit to Meta → deliver on approval (usually minutes) → **auto-delete ~24h later**. If an approved template already matches the content, it's reused instantly.

| Content type | Window closed behavior | Status |
|---|---|---|
| **Text** | One-off text template → sends on approval → reopens chat | ✅ **Already built** |
| **Image** | One-off **image-header** template (image headers exist for campaigns, not yet wired to forward) | ⚠️ **New build** |
| **Document** | One-off **document-header** template — Meta supports it; our system doesn't yet | ⚠️ **New build** |
| **Video** | One-off **video-header** template — Meta supports it; our system doesn't yet | ⚠️ **New build** |
| **Audio** | No audio template header exists on WhatsApp | ❌ **Impossible** (Meta limit) |

> Correction worth noting: **document and video are NOT a Meta wall** — Meta supports those template headers; we simply only wired up IMAGE (hardcoded). So they're buildable. **Audio is the only true dead-end.**

**Media delivery shape:** the file rides in the template **header**; a short **agent-editable line** becomes the body (e.g. "Hi 👋 here's the file you asked about."). Customer gets the file + that line — it **replaces the original caption**.

**Agent experience:** Picks any destination. Into a closed window, text/image/document/video show "Queued · reopens chat — delivers once WhatsApp approves." Audio is blocked with a reopen workaround.

**Effort:** Text = none (exists). Image = low-medium (header exists). **Document + video = medium** (net-new header support in template create/send + media reopen path). Audio = N/A.

**Pros:** Agent genuinely reaches quiet conversations with the actual content; reuses proven one-off machinery. **Cons:** Not instant (approval wait, usually minutes); consumes template quota; caption replaced by the body line; more to build.

---

## Side by side

| | Block | Create templates |
|---|---|---|
| Text post-24h | 🚫 blocked | ✅ reopens (exists) |
| Image post-24h | 🚫 blocked | ⚠️ reopens (new build) |
| Document / Video post-24h | 🚫 blocked | ⚠️ reopens (new build, buildable) |
| Audio post-24h | 🚫 blocked | 🚫 impossible (Meta limit) |
| Delivery speed | n/a | Minutes (approval) |
| Build effort | Low | Low (text/image) → Medium (doc/video) |
| Reaches quiet conversations | ❌ no | ✅ yes (except audio) |

---

## ✅ Approved direction — Create templates (text + image + document + video)

Decided with product: forwarding into a closed window **reopens the chat via a one-off template** for **text, image, document, and video**. The file rides in the header with a short **agent-editable line** as the body. **Audio is the only blocked type** (no Meta audio header) — surfaced with a reopen workaround + "notify me when they reply."

This is what the mockup now demonstrates: text/image/document reopen via one-off; audio blocked.

## Remaining decisions / follow-ups
1. **Build sequencing:** text + image first (least new work), then document + video headers — ship together or staged?
2. **Default body line** per type, and may the agent leave it blank? (Meta requires a body for media templates.)
3. **Rejection path:** if Meta rejects a one-off on content policy, how is the failed forward shown + retried?
4. **Audio:** is "notify me when they reply" a real capability or just a message?
