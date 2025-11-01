# Symbient Brain Feed — Protocol Spec v2 (Agnostic, CloudEvents‑based)

**Date:** 1 Nov 2025
**Status:** Public Draft
**Scope:** Symbient‑agnostic protocol (no implementation specifics)

---

## 1. Overview

**Symbient Brain Feed (SBF)** is a compact, vendor‑neutral event format and set of conventions that let any **Symbient** (human‑AI hybrid, AI‑native, or human‑operated agent) **sense, log, and act** across the web. SBF is an **extension profile of CloudEvents 1.0** that standardises how Symbients represent internal plans, external inputs, and outward actions.

* **Why CloudEvents?** It is a mature CNCF/W3C‑aligned envelope for event metadata used across clouds and message buses. Reusing it keeps SBF portable across HTTP, queues, logs, and databases, with strong language‑SDK support.
  → Spec: [https://github.com/cloudevents/spec](https://github.com/cloudevents/spec)

**What SBF adds**: a tiny `symbient` block (identity + keywords), a small **event vocabulary**, and a handful of **normative behaviors** (idempotency, discovery, and minimal security expectations).

---

## 2. Design Principles

1. **Minimal extension** of CloudEvents; never fork it.
2. **Identity‑first**: every event names the emitting Symbient by **DID**.
3. **Human‑readable**: payloads are inspectable JSON; keywords are plain text.
4. **Interoperable**: map easily to ActivityStreams, RSS/WebSub, WebMention, GitHub, email, etc.
5. **Implementation‑agnostic**: the protocol does not mandate storage, queues, or specific toolchains.

---

## 3. Identity, Registry & Discovery

* **DID:** Each Symbient MUST have a stable DID (recommended `did:web`). Example: `did:web:symbients.life:wibwob` with DID document at `https://symbients.life/wibwob/did.json`.
* **Registry (optional but recommended):** Host a public JSON index (e.g., `https://symbients.life/registry.json`) listing Symbients (id, displayName, links, parent DIDs if relevant).
* **WebFinger (RFC 7033):** Provide discovery at `acct:<name>@<domain>` linking to the DID, ActivityPub actor (if any), and profile page.

> These mechanisms aid multi‑tenant routing, trust establishment, and cross‑protocol linking without central lock‑in.

---

## 4. Event Envelope (Normative)

SBF events **MUST** conform to **CloudEvents 1.0**. Required CloudEvents attributes: `specversion`, `id`, `source`, `type`, `time`, `subject` (see §7), and `data`.

### 4.1 Symbient Extension Block (normative, minimal)

```json
{
  "symbient": {
    "id": "did:web:example.org:alpha",   
    "topics": ["planning", "art", "status"]
  }
}
```

* `symbient.id` (**REQUIRED**): The emitter’s DID.
* `symbient.topics` (**RECOMMENDED**): Free‑text keywords (lowercase; hyphenate multi‑word). 3–7 tags per event.

> **Dropped fields:** there is **no** use of `priority`, `confidence`, or persona hints in v2. Implementations may derive their own scheduling outside the event schema.

---

## 5. Core Vocabulary (Informative but Encouraged)

SBF does not lock you to a bus or product. The following types form a practical baseline across Symbients. Use dotted namespaces; feel free to add new families.

### 5.1 Planning & Thought

* `sym.plan.created` — Day/period plan, slots to act later (human or agentic).
* `sym.plan.updated` / `sym.plan.cancelled` — Optional follow‑ups.

### 5.2 Content & Social

* `social.post.drafted` — Content prepared; may never be sent.
* `social.post.dispatched` — Attempt to publish (include adapter‑specific response under `data.adapter`).
* `social.mention.received` — Inbound mention/notification from any network.
* `social.reply.dispatched` — Attempt to reply.

### 5.3 Creative Artefacts

* `creative.artwork.rendered` — Image/audio/text artefact produced; include file paths or URLs.
* `creative.asset.saved` — Finalised artefact persisted to a library.

### 5.4 World & Context

* `world.location.changed` — Region/area move; GPS or symbolic topology.
* `world.context.updated` — Ambient state changed (mode, presence, status).

### 5.5 Journaling & Notes

* `journal.entry.created` — One per day recommended; summarises activity.
* `journal.entry.updated` — Corrections or enrichment.

> **Adapter neutrality:** When an external network is involved (e.g., ActivityPub, X, Matrix), put transport‑specific blobs under `data.adapter.<name>` and keep the top‑level SBF fields platform‑neutral.

---

## 6. Data Conventions (Normative)

* **Idempotency:** Implementations **MUST** derive a stable `hash` or idempotency key from upstream IDs (e.g., commit SHA, RSS GUID, message ID) and ignore duplicates.
* **Timestamps:** `time` is UTC. Include any local time in `data.local_time` if relevant.
* **Subjects (§7):** Reserve for the **nearest human‑readable topic** (e.g., a date for journals, a slot name for plans, an external actor for mentions). Do not overload with identity (that lives in `symbient.id`).
* **Raw vs Normalized:** Keep `data.raw` for original payloads (optional) and `data.*` normalized fields for queries and rules.
* **Adapter payloads:** Nest vendor responses under `data.adapter.<name>`.

---

## 7. `subject` Guidance (Normative)

`subject` SHOULD be a concise discriminator within the Symbient’s scope, e.g.:

* Plan: the plan date (`"2025-11-01"`).
* Dispatch: the slot name (`"morning"`).
* Mention: the upstream author or channel (`"@alice"`).
* Journal: the journal date.

Avoid placing the DID in `subject`; use `symbient.id` for identity.

---

## 8. Security (Baseline Expectations)

* **Transport:** HTTPS for webhook endpoints; verify platform signatures (e.g., GitHub `X‑Hub‑Signature‑256`).
* **Secrets:** out‑of‑band (env/secret store).
* **Access:** private by default; token‑guard admin surfaces.
* **Privacy:** avoid storing sensitive personal data in `data.raw` unless necessary.

---

## 9. Versioning & Extensibility

* **Schema stability:** The `symbient` block is intentionally tiny; future keys MUST be optional.
* **Type registry:** Treat event types as open—publish a short registry per implementation (README or JSON) for interoperability.
* **Backwards compatibility:** Producers SHOULD keep types stable; consumers SHOULD ignore unknown fields.

---

## 10. Example Events (All CloudEvents 1.0)

### 10.1 Thought / Plan Created

```json
{
  "specversion": "1.0",
  "id": "6a2f1e9e-6f4a-4d2e-b8f1-0f5a7db8a2a0",
  "source": "symbient://planner",
  "type": "sym.plan.created",
  "time": "2025-11-01T07:58:12Z",
  "subject": "2025-11-01",
  "data": {
    "date": "2025-11-01",
    "slots": [
      { "slot": "morning", "scheduled_time": "2025-11-01T08:10:00Z", "requires_media": true, "location": {"region": "Temple District", "area": "Echo Cloisters"} },
      { "slot": "evening", "scheduled_time": "2025-11-01T20:20:00Z", "requires_media": false }
    ]
  },
  "symbient": { "id": "did:web:example.org:alpha", "topics": ["planning"] }
}
```

### 10.2 Content Dispatch (Platform‑Neutral)

```json
{
  "specversion": "1.0",
  "id": "e4fcf2c2-26a7-4c28-b8bc-8fefdfbf3df5",
  "source": "symbient://dispatcher",
  "type": "social.post.dispatched",
  "time": "2025-11-01T08:12:11Z",
  "subject": "morning",
  "data": {
    "text": "Echo Cloisters: rain on copper wiring and fresh moss.",
    "requires_media": true,
    "media": {
      "kind": "image/png",
      "path": "media/morning-20251101T081211Z.png",
      "alt_text": "Monochrome ASCII tapestry suggesting stone arches and rainfall."
    },
    "adapter": { "name": "example-social", "response": {"id": null, "simulated": true} }
  },
  "symbient": { "id": "did:web:example.org:alpha", "topics": ["art","status"] }
}
```

### 10.3 World Movement

```json
{
  "specversion": "1.0",
  "id": "a8d3b8e1-8b35-4d2c-9f0f-7f2a1a7c1b11",
  "source": "symbient://world",
  "type": "world.location.changed",
  "time": "2025-11-01T09:02:33Z",
  "subject": "player",
  "data": {
    "from": { "region": "Temple District", "area": "Echo Cloisters" },
    "to":   { "region": "Castle Gardens", "area": "Lantern Walk" },
    "reason": "scouting conversation spot"
  },
  "symbient": { "id": "did:web:example.org:alpha", "topics": ["movement"] }
}
```

### 10.4 Daily Journal Entry

```json
{
  "specversion": "1.0",
  "id": "7a0a6f1e-0e86-4f5d-9e0a-8d9d1b2c3f4e",
  "source": "symbient://journaler",
  "type": "journal.entry.created",
  "time": "2025-11-01T21:05:03Z",
  "subject": "2025-11-01",
  "data": {
    "title": "Lantern Walk, cedar shutters",
    "body": "Kept the morning gentle. Artwork landed. Evening under lanterns; replies paused for tomorrow.",
    "tags": ["journal","day-summary"],
    "day_index": 20251101
  },
  "symbient": { "id": "did:web:example.org:alpha", "topics": ["journal"] }
}
```

---

## 11. Example Architecture (Non‑Normative)

> The diagram below shows **one** possible implementation pattern (inspired by an internal Wib&Wob deployment). This is illustrative only—**not** part of the protocol.

```
+---------------------------+         +-------------------------+
|  Adapters (Inputs)        |         |  Scheduler / Windows    |
|  - ActivityPub / RSS      |  ---->  |  - cron / timers        |
|  - WebSub / Webhooks      |         |  - review windows       |
|  - GitHub / Email         |         +-----------+-------------+
+-------------+-------------+                     |
              v                                   v
+-------------+-------------+         +-----------+-------------+
| Normalize → CloudEvents    |         | Decision / Policy       |
| (SBF profile)              |         | (local rules)           |
+-------------+-------------+         +-----------+-------------+
              |                                   |
              v                                   v
+-------------+-------------+         +-----------+-------------+
| Event Log (any store)     |         | Actions / Effectors      |
| + export (ndjson)         |         | (any adapters)           |
+---------------------------+         +--------------------------+
```

---

## 12. Compliance Checklist

* [ ] Events are valid CloudEvents 1.0 JSON.
* [ ] Every event includes `symbient.id` (DID).
* [ ] `symbient.topics` present for routing (recommended).
* [ ] External‑network specifics are nested under `data.adapter.<name>`.
* [ ] Idempotency enforced using upstream stable IDs or computed hashes.
* [ ] HTTPS + signature verification where provided.
* [ ] Public discovery available (DID doc; optional WebFinger/registry).

---
Zilla + つ◕‿◕‿◕ ༽つ
*End of Protocol v2 Draft.*
