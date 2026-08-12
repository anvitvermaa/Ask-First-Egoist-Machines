# Ask First — Creative Pass for AI Passport

**🔗 Live demo: [ask-first.netlify.app](https://ask-first.netlify.app/)**

**AI Passport Ideathon · Track: Creative · Lane: Build · Solo entry: Arun Verma**

> Consent should be resolved, not declared once.

A creator's honest answer to AI is **nine independent decisions**, not one — and it depends on **who is asking**. Opt-out standards have started closing the granularity gap. None of them close the two that matter: the answer is still read at crawl time by whoever bothers, and once read it cannot be withdrawn.

Ask First moves the answer onto the Egoist AI Passport as a **Creative Pass** — standing policy per use-type *and* per requester class, fetched by registered requesters at the moment of use, returning a short-lived lease instead of a licence.

---

## What's here

| File | What it is |
|---|---|
| `index.html` | Working prototype. Single file, no build step, no dependencies, no network calls. |
| `SPEC.md` | Protocol spec — enforcement model, wire formats, decision order, threat model, rollout. |
| `Ask-First-onepager.pdf` | Two-page visual brief. |

---

## Run it

Open `index.html` in any browser. No install, no server, no keys.

Press **Start** for the guided walkthrough, or drive it yourself.

### The demo path

1. **StemForge AI** asks to generate in Juno Kade's style, non-commercially → **allow**, lease minted with a live countdown.
2. Same company, same use, now for an ad campaign → **deny**. The use is fine; the commerce isn't. The refusal lands in the demand inbox as a priceable offer.
3. **OpenMusic Lab (research)** asks to train on her catalogue → **escalate** to the Passport Inbox as a proposal.
4. **Adhouse Agency (commercial)** asks for the identical thing → **deny**, and it never reaches her Inbox. *Who is asking is part of the policy — the dimension every content-anchored standard collapses.*
5. **An unregistered crawler** tries → gets nothing it can learn from. Requester class is asserted against a credential, never self-declared.
6. Flip `style-transfer` to **deny** in the policy panel.
7. The identical request from step 1 now resolves **deny** — nothing recalled, nothing deleted, because the answer was never copied out in the first place.

Leases run on a compressed **20-second TTL** so you can watch one expire. Real TTLs would be minutes to a session.

---

## The mechanism

Published work carries a **pointer** (`X-Creative-Passport: ego:junokade`), not a licence. The pointer is inert — all authority stays at the Passport. A **registered** requester calls `POST /creative/resolve` at the moment of use and gets `allow` / `deny` / `escalate`. Class comes from the credential, not the request body. An `allow` returns a lease: a bearer handle with a hard expiry, re-verified before each covered act, killable mid-flight. Every outcome writes a **content-free** receipt — with the reason detail going to the creator's log and only an opaque code to the requester, so the policy can't be reconstructed by probing.

---

## What it enforces, and what it only makes legible

**This is the first question anyone serious will ask, so it's answered up front.** Consent protocols are either *custody-based* (the intermediary holds the asset and can withhold it) or *declaration-based* (it holds nothing, and makes the permission state legible and non-repudiable).

Creative Pass is **declaration-based for public work**, because being custody-based would mean Egoist holding every creator's catalogue — a concentration risk worse than the problem. So a lease does not stop a party that already scraped a track. It changes what that party can later claim: "the opt-out was ambiguous, we didn't know" stops being available when there was a machine-readable answer at a known address and the record shows you were refused or never asked.

It is **custody-based where custody actually exists** — voice models, likeness reference, unreleased stems, high-res masters. There the lease genuinely gates the bits, and revocation bites immediately. That's also where the harm is worst, which is why V1 starts there.

And the limit nothing fixes: **this does not un-train a model.** No consent layer can.

---

## Where it sits next to existing work

IETF AI-preferences, RSL, C2PA data-mining assertions and Cloudflare pay-per-crawl are all real and all useful. Every one is anchored to **content or origin** and read **once, at fetch**. Creative Pass is anchored to a **person** and read **at use** — which is what makes withdrawal mean something, makes per-requester answers possible, and produces a receipt the creator owns. It composes with all four; the pointer can ride inside a C2PA assertion.

---

## Built on primitives Egoist already has

| Egoist primitive | How Creative Pass uses it |
|---|---|
| **Pass** (app × category × duration) | Extended to requester × use-type × class × commercial posture × duration |
| **Inbox / proposal** | Where out-of-policy requests escalate, with duplicate collapse |
| **Protected memory** | Proposed default tier for `voice-synth` and `likeness` |
| **Content-free receipt** | Written on every outcome, including refusals |

New surface required: a credentialed resolver, requester registration, and a lease service. Stated plainly rather than implied away.

---

*Prototype for the Egoist Machines AI Passport Ideathon. Not affiliated with Egoist Machines, Inc. Endpoints under `passport.ego.ist/creative/*` are proposed, not live. Juno Kade is a fictional profile from Egoist's own landing page, used here as a demo persona.*
