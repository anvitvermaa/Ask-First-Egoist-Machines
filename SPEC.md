
# Creative Pass — Protocol Specification

 

**Draft 0.2 · AI Passport Ideathon · Track: Creative · Lane: Build**

 

A use-time consent protocol for creative work, specified as an extension of the Egoist AI Passport `pass` primitive.

 

This document covers mechanism: wire formats, decision order, enforcement model, threat model. The case for the idea is in `SUBMISSION.md`; the running implementation is `index.html`.

 

---

 

## 0. Status, and what this is not

 

**Status:** design draft with a working reference implementation. Endpoints under `passport.ego.ist/creative/*` and the discovery document are **proposed**, not live.

 

**Non-goals**, stated up front because each is a claim a reader would otherwise assume:

 

| Non-goal | Why |

|---|---|

| Un-training a model | Not achievable by any consent layer. Weights that absorbed a work are not reachable by a revocation. |

| Physically preventing use of already-public work | See §3. This protocol is declaration-based for public work and custody-based only for non-public reference material. Conflating the two is the most common way this class of system oversells itself. |

| Detecting infringement | Not a content-matching or watermarking system. Composes with those; does not replace them. |

| Adjudicating ownership | Records claims and decisions. Does not decide who owns what. |

| **Being a corpus host** | Egoist must never hold the creative works themselves. Holding permissions is defensible; holding everyone's unreleased catalogue is a liability that dwarfs the problem being solved. |

 

On work identifiers: §5 defines an optional `work=` qualifier. These are **creator-local labels stored on the creator's own Passport**, used to scope policy to a single piece. They are not a global registry, are not enumerable by third parties, and the protocol is fully functional without them — policy then applies at Passport level, which is what the reference implementation does. **Per-work policy overrides are declared here but deliberately not implemented in the prototype**, since Passport-level policy is sufficient to demonstrate every mechanism that matters and per-work overrides are a straightforward specialisation of it.

 

---

 

## 1. Prior art, and the actual delta

 

The field is not empty. Anything claiming otherwise should be distrusted.

 

| Existing work | What it does | Where it stops |

|---|---|---|

| IETF AI-preferences | Standardising vocabulary beyond a single crawler bit | Expressed in a file at an origin; read at fetch; no withdrawal channel |

| RSL | Machine-readable licensing terms attached to content | Anchored to the content, not the person; static once published |

| C2PA / content credentials | Provenance, plus data-mining assertions | Backward-looking: establishes origin, not forward permission; travels in metadata that gets stripped |

| Cloudflare pay-per-crawl | A toll booth in the network path | Gates fetch, not use; only works for content behind that network |

 

All four are anchored to **content or origin** and read **once, at fetch**.

 

**The delta:** Creative Pass is anchored to a **person** and read **at use**. Three consequences that none of the above can produce:

 

1. **Withdrawal means something**, because no requester holds a copy of the answer.

2. **Per-requester answers become possible**, because the decision is computed at request time against a known principal — a research lab and an ad agency get different answers to the same question.

3. **The creator gets a receipt**, including for refusals, which is where the demand signal lives.

 

It composes with all four. The pointer in §5 can ride inside a C2PA assertion.

 

---

 

## 2. Terminology

 

Carried from AI Passport with existing meaning: **Passport** (`ego:<username>`), **receipt** (content-free activity record), **protected memory** (a tier above ordinary approval).

 

Borrowed with a stated change of object: **Inbox / proposal**. Egoist's Inbox reviews incoming *memories* before they become approved memory. Creative Pass reuses the same surface and review gesture for incoming *access requests*. Same UI, different object — an extension, not a carry-over.

 

**Extended by this spec — stated explicitly, because it is the load-bearing change:**

 

> Egoist's **Pass** is scoped to *one connected app × one memory category × one duration*. A **Creative Pass** is scoped to *one registered requester × one use-type × one requester class × one commercial posture × one duration*.

 

Substituting "registered requester" for "connected app" is not cosmetic: it widens the counterparty set from apps the user personally connected to arbitrary parties on the internet. §4 exists because of that widening.

 

**New terms:** **use-type** (§3) · **resolution** (§6) · **lease** (§7) · **standing policy** (§5).

 

---

 

## 3. Enforcement model

 

The first question any serious reader asks: *the requester already has the file — what does a lease actually gate?*

 

Consent protocols fall into two families.

 

**Custody-based.** The intermediary holds the asset and can withhold it. Enforcement is physical. Cost: the intermediary must hold everyone's work, which is precisely the concentration risk this design refuses (§0).

 

**Declaration-based.** The intermediary holds no asset. It makes the permission state legible, timestamped, addressable and non-repudiable.

 

**Creative Pass is declaration-based for public work.** A lease does not stop a party that already scraped a track. What it changes is what that party can subsequently claim. Today the standard defence is ambiguity — the opt-out was unclear, the signal was missing, we did not know. That defence stops being available when there was a machine-readable answer at a known address and the record shows the party either resolved and was refused, or never asked at all. It moves the dispute from contested facts to settled ones.

 

**It is custody-based where custody actually exists.** Voice models, likeness reference, unreleased stems and high-resolution masters are not public. For those, reference material is delivered *through* the lease, the lease genuinely gates the bits, and revocation bites immediately. This is also where the harm is most severe — which is why V1 (§12) starts there rather than with text scraping.

 

Being precise about which half applies where is the difference between a protocol and a press release.

 

**"But the custody half is already a bilateral contract — why does it need a protocol?"** The natural next objection, and the answer is operational rather than legal. A signed licence cannot be revoked mid-generation. It cannot be scoped to `style-transfer` but not `voice-synth`, then changed six months later without re-papering. It does not survive the artist's counterparty being acquired. And it does not scale: a vendor onboarding ten thousand artists cannot negotiate ten thousand agreements, which is precisely why vendors today either license a famous handful or scrape everyone else. A resolver is what turns a bespoke deal into an operation. And when the enterprise buyer asks for evidence, **the artifact they can actually check is the receipt, not the contract** — a contract asserts a right existed in the abstract; a receipt shows this specific generation was authorised at the moment it happened.

 

**On "provably unconsented":** the claim holds against an *enrolled* requester, whose receipt log will show either a refusal or a gap. It does **not** hold against a party that never enrolled — absence of a receipt from a non-participant is unobservable. The protocol strengthens the record for participants; it does not manufacture evidence about non-participants.

 

---

 

## 4. Requester registration

 

Requesters are **registered principals with credentials**. `requester_class` is asserted by Egoist against the registration, never read from the request body.

 

This is not optional hardening. If class were self-declared, an ad agency would send `requester_class: "research"` and the entire per-class policy dimension would be theatre.

 

```http

POST /creative/register

→ { "client_id": "stemforge.ai", "class": "commercial", "verified_at": "...", "secret": "..." }

```

 

Registration requires a verifiable business identity, and class changes are re-verified.

 

**Unregistered callers are rejected at the transport layer: `401`, empty body.** No decision, no receipt identifier, and no confirmation that the Passport exists — returning any of those would disclose the same thing `404 passport_not_found` exists to conceal. `requester_not_registered` is therefore an **HTTP error, not a decision code**; it never appears in the decision vocabulary of §9.

 

**Protected-tier clearance is a second, higher bar.** `voice-synth` and `likeness` require a principal separately audited and cleared for protected access — being a known company is not the same as being trusted with someone's voice. A registered-but-uncleared requester resolves normally on every other use-type and is refused on these two.

 

**Trade-off, stated plainly:** registration is friction, and friction suppresses adoption. The answer is that the requesters who matter commercially *want* to be registered, because the receipt is what they show their client (§11). A protocol optimised for the anonymous scraper's convenience would have no value to anyone else.

 

---

 

## 5. Standing policy, binding, discovery

 

### Policy object

 

Held on the Passport. **Never returned to requesters** — see §9.

 

```json

{

  "passport": "ego:junokade",

  "default_unknown": "ask",

  "policy": {

    "index":          { "mode":"allow", "commercial_ok":true,  "attribution":false,

                        "classes":["research","commercial","individual","platform"] },

    "quote":          { "mode":"allow", "commercial_ok":true,  "attribution":true,

                        "classes":["research","commercial","individual","platform"] },

    "retrieve":       { "mode":"allow", "commercial_ok":true,  "attribution":true,

                        "classes":["research","commercial","individual","platform"] },

    "train:general":  { "mode":"ask",   "commercial_ok":false, "attribution":true,

                        "classes":["research"] },

    "train:finetune": { "mode":"deny" },

    "style-transfer": { "mode":"allow", "commercial_ok":false, "attribution":true,

                        "classes":["research","commercial","individual","platform"] },

    "remix":          { "mode":"ask",   "commercial_ok":false, "attribution":true,

                        "classes":["research","individual"] },

    "voice-synth":    { "mode":"deny",  "tier":"protected" },

    "likeness":       { "mode":"deny",  "tier":"protected" }

  }

}

```

 

`mode` ∈ `allow` · `ask` · `deny`. `classes`, `commercial_ok`, `attribution` and `max_ttl` are conditions evaluated per §6.

 

### Use-type taxonomy

 

Nine independent decisions, in three families. The design claim is that a creator's honest answer differs across all nine, and that collapsing them is what makes existing tooling unusable.

 

**Reading** — `index`, `quote`, `retrieve`

**Learning** — `train:general`, `train:finetune`

**Generating** — `style-transfer`, `remix`, `voice-synth`, `likeness`

 

Most creators want `index` (that is distribution) and refuse `voice-synth` (that is impersonation). A protocol that cannot express *find me, but do not become me* is not describing consent.

 

Use-types are namespaced strings. Unknown use-types resolve to `default_unknown`, which **must** default to `ask`. Silence is never a yes.

 

### Binding

 

A published work carries a pointer, not a licence:

 

```

X-Creative-Passport: ego:junokade; work=wrk_3f81a2

```

 

Also expressible as an HTML `<meta>`, an ID3/XMP field, or a C2PA assertion. The pointer is inert — it carries no permissions. All authority stays at the Passport and is fetched live. This is what lets a change of mind reach copies already in circulation: the copy never carried the answer, only the address of the person who has it.

 

### Discovery

 

Discovery resolves from the **Passport identifier in the pointer**, not from the hosting origin:

 

```

GET https://passport.ego.ist/.well-known/creative-pass

```

 

Origin-anchored discovery would break on exactly the case this protocol exists for — a work re-hosted somewhere the creator does not control.

 

---

 

## 6. Resolution

 

### Request

 

```http

POST /creative/resolve

Authorization: Bearer <client_credential>

```

```json

{

  "passport": "ego:junokade",

  "work": "wrk_3f81a2",

  "use": "style-transfer",

  "commercial": true,

  "purpose": "backing track for client campaign",

  "duration": "session"

}

```

 

`requester` and `requester_class` are **not** accepted from the body. They are derived from the credential (§4). `purpose` and `duration` mirror the field-request shape on Egoist's developer surface.

 

### Decision order

 

Normative, and implemented in `index.html`:

 

0. Credential absent or invalid → **`401`, empty body.** Not a decision; the request never reaches the engine.

1. `mode == "deny"` → `deny` / `refused_by_policy`

2. Requester class not in `classes` → `deny` / `refused_by_policy` *(offerable if commercial)*

3. `tier == "protected"` and principal not cleared for protected access (§4) → `deny` / `refused_by_policy`

4. `mode == "ask"` → `escalate` / `pending_creator_review`

5. `commercial == true` and `commercial_ok == false` → `deny` / `refused_by_policy` *(offerable)*

6. Otherwise → `allow` / `granted`

 

Step 4 precedes step 5 deliberately. Conditions qualify an `allow`; they must not silently convert a request the creator asked to review into a refusal she never saw.

 

### Responses

 

**allow**

```json

{ "decision":"allow", "code":"granted",

  "requester":"stemforge.ai", "requester_class":"commercial",

  "requester_auth":"client_credential:verified",

  "lease": { "id":"lease_8f2ad10c", "expires_in_seconds":900,

             "obligations":["attribute:@junokade","no-redistribute-reference","no-sublicense"],

             "reverify_before_each_use": true },

  "receipt": { "id":"rcpt_71b0c9de", "content_hash": null } }

```

 

**deny**

```json

{ "decision":"deny", "code":"refused_by_policy",

  "terms_available": true, "next":"POST /creative/offer",

  "receipt": { "id":"rcpt_44de81a0", "content_hash": null } }

```

 

**escalate**

```json

{ "decision":"escalate", "code":"pending_creator_review",

  "proposal": { "id":"prop_9c22ff81", "inbox":"<the creator's Passport Inbox>",

                "status":"pending_creator_review", "times_asked": 3 },

  "note":"No context released while pending. Silence is a denial, not a default yes.",

  "receipt": { "id":"rcpt_02aa7fe3", "content_hash": null } }

```

 

### Escalation and duplicate collapse

 

Standing policy answers the routine case silently. Only out-of-policy requests escalate, and **repeat asks for the same (requester, use-type) pair collapse into a single Inbox proposal with a counter** rather than generating one item per request. Without collapse, a thousand labs asking for `train:general` produce a thousand notifications, and a consent system that fires a thousand notifications has not obtained consent — it has manufactured a reflex.

 

If a creator is fielding more than a handful of proposals a week, the defaults are wrong. That is a product bug, not a user problem.

 

---

 

## 7. Lease semantics

 

A lease is not a licence and not the permission. It is a bearer handle with three properties:

 

1. **Expires by construction.** No renewal, no authority. Minutes for generation; up to a session for retrieval.

2. **Re-verified at each use**, not once at acquisition. `POST /creative/lease/verify → { "valid":false, "state":"revoked" }`

3. **Killable mid-flight.** Revocation invalidates immediately.

 

Why a handle rather than the permission: **a copy of a permission cannot be taken back; a handle can be switched off.** That is the whole mechanical difference between revocation-as-promise and revocation-as-mechanism.

 

Requesters must not cache outcomes beyond lease expiry. Verification is served by Egoist, so a lease is verifiable by the creator, the requester, and any party either of them shows it to — but **not** independently of Egoist. Cryptographically signed decisions (allowing offline verification against a published key) are the obvious v0.3 addition and are not claimed here.

 

---

 

## 8. Receipts

 

Written on every outcome, including refusals and escalations. Owned by the **creator**.

 

```json

{ "id":"rcpt_44de81a0", "ts":"2026-08-12T02:41:07Z",

  "requester":"adhouse.agency", "use":"voice-synth", "decision":"deny",

  "content_hash": null,

  "note":"content-free: no prompt, output or asset recorded" }

```

 

Receipts **must** be content-free. A receipt carrying prompts or outputs would make the consent layer the most invasive log on the internet — a complete record of everything anyone ever tried to generate about anyone.

 

---

 

## 9. Response asymmetry

 

The requester receives an **opaque code**. The creator's receipt receives the **reason detail**.

 

This matters: returning `"reason": "policy.style-transfer.commercial_ok = false"` to the caller hands them the policy path and its value. Roughly eighteen calls reconstructs the entire policy object — which is exactly the enumeration attack §10 claims to defend against. Codes are drawn from a fixed set of three — `granted`, `refused_by_policy`, `pending_creator_review` — and carry no policy structure. Notably, class refusal, protected-tier refusal and commercial refusal all return the identical `refused_by_policy`, so a requester cannot tell *which* dimension refused them. The creator's receipt records exactly which.

 

---

 

## 10. Errors

 

Transport errors and decisions are strictly separate. A decision always returns `200`; anything non-`200` means no decision was made.

 

| Status | Meaning |

|---|---|

| `200` + `decision:"escalate"` | Unknown use-type falls through to `default_unknown`. An unrecognised use is escalated, never errored — a 4xx invites the caller to proceed as if unanswered |

| `401 requester_not_registered` | No valid credential. Empty body (§4) |

| `403 no_pointer` | Work not bound to a Passport |

| `404 passport_not_found` | No such Passport, or not published |

| `409 lease_revoked` | Lease killed |

| `410 lease_expired` | TTL elapsed |

| `429 probe_limit` | Enumeration defence (§11) |

 

---

 

## 11. Threat model

 

**Requester impersonation.** The most obvious attack: claim to be a research lab, get the research-lab answer. *Mitigation:* §4 — class derived from a verified credential, never from the request body. This is why registration is mandatory rather than convenient.

 

**Enumeration.** Brute-forcing resolutions to map a creator's policy. *Mitigation:* §9 response asymmetry; per-credential rate limits (`429`); repeated refused probes surfaced to the creator as a pattern rather than as noise.

 

**Concentration risk.** Consent for millions of creators in one place. *Mitigation:* §0 — permissions and pointers only, never the corpus. A breach leaks preferences, not everyone's unreleased catalogue. Genuinely bad; not catastrophic.

 

**Creator-side surveillance.** Every resolution reveals interest. *Mitigation:* §8 content-free receipts, creator-owned; requesters cannot query the log.

 

**Requester-side confidentiality.** Less obvious and equally real: the resolver log is a map of every lab's unreleased product intentions. A vendor resolving `voice-synth` against forty artists has disclosed a roadmap. *Mitigation:* resolution volume is visible to the creator being asked and to no one else; Egoist publishes no cross-requester analytics and must contractually forgo them. An intermediary that monetised this signal would deserve to lose the market.

 

**Consent fatigue.** The failure mode of every consent system shipped to date. *Mitigation:* standing policy, escalation only for out-of-policy, and duplicate collapse (§6).

 

**Coercion by distribution.** A platform conditions distribution on an over-broad pass. *Mitigation:* passes carry a `coerced` flag; conditioning access should be a terms violation. **This is a governance answer, not a technical one.** The protocol can surface coercion; it cannot prevent it.

 

**Co-ownership.** A master is typically co-owned with a label, publishing split with a PRO. A single Passport holder is often not the sole rightsholder. *Mitigation:* a work may carry multiple pointers; where it does, `allow` requires **all** bound Passports to allow, and any one may refuse. Refusal is unanimous-optional; permission is unanimous-required. Delegation (label acts for artist within stated bounds) is expressed as a scoped delegation pass. This is genuinely underspecified here and is the first thing a music-industry reviewer would press on.

 

**False binding.** Someone claims work that is not theirs. Binding occurs at publication from an authenticated Passport; disputes are handled out of band.

 

**Retention and jurisdiction.** Receipts are personal data under GDPR. Creator-facing receipts should be retained as long as the Passport lives and be exportable and erasable on request; requester-facing decision records are business records with a shorter, stated retention. Creators in the EU have Article 17 and Article 20 rights over their own receipt log, and the log must be built assuming that from day one rather than retrofitted.

 

**And the limit that no mitigation reaches:** this does not un-train a model. Use-time resolution buys a bounded window, not a rewind.

 

---

 

## 12. First version

 

**Not the frontier labs, and not a standards coalition** — that is how every consent standard has died. Also not "the assistant checks before generating": MCP tools are model-discretionary, the connector is installed by the *prompting user* rather than the creator, and a mandatory pre-generation hook would require Anthropic, OpenAI and Google to each ship one. That is a coalition of the three hardest parties to convene, wearing a disguise.

 

**Start where consent is already a sales blocker.**

 

1. **One voice or likeness platform.** These vendors cannot close enterprise deals without consent provenance — agencies and brands demand indemnification, and competitors already market on licensed-and-indemnified data. They integrate because their sales cycle requires it, not out of goodwill. This is also the custody-based half (§3), where the lease genuinely gates the bits. Ship three use-types: `voice-synth`, `likeness`, `style-transfer`.

2. **Creators onboard where they already are** — the public page at `ego.ist/i/you` gains a consent panel. Nine tri-state controls with defaults set, not a blank form.

3. **Publishing tools next.** The pointer is one header; adoptable without buying into anything else.

4. **Assistants after that**, correctly framed: the Passport connector lets a creator's policy be *surfaced* to an assistant, which is advisory, not enforcement. Useful, and not to be oversold.

 

**Who pays:** requesters, for verified consent provenance they can put in a contract. Creators pay nothing, consistent with the Passport's free-for-life consumer promise. The anonymous scraper never joins and does not need to — the money is at the top of the market, not the bottom.

 

**90-day success:** one voice platform resolving in production, a few thousand creators with non-default policy, and real priced offers in demand inboxes. The offers are the proof the market exists.

 

---

 

*Reference implementation: `index.html` — standing policy per use-type and per requester class, credentialed resolution, the decision order above, expiring leases, duplicate-collapsing escalation, asymmetric content-free receipts, and a revocation that flips an identical request from allow to deny.*
