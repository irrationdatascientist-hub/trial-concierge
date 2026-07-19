# Trial Concierge — Case Study Scaffold

*A deployment case study for the ElevenLabs Deployment Strategist application. Fill the bracketed prompts with your own words and numbers. Keep it to ~2 pages + the demo link. Write it the way you'd frame a real customer engagement — that framing IS the audition.*

---

## 1. One-line summary

> A production-shaped ElevenAgents voice + chat assistant that lets clinical research associates get instant, protocol-grounded answers on eligibility, visit schedules, and endpoints — reducing screening failures and protocol deviations.

[Rewrite in your own voice. Lead with the outcome, not the tech.]

---

## 2. The workflow and the pain point

*Why this problem, and why it matters commercially. This section is where your domain credibility shows — write it as someone who's worked inside clinical-trial operations.*

- **The workflow:** [Describe how a CRA / site coordinator actually works a protocol today — flipping through a 100+ page document, emailing the medical monitor, waiting on answers, during a live patient visit.]
- **Where it breaks:** [Misread inclusion/exclusion criteria → screening failures. Missed visit windows → protocol deviations. Ambiguity → costly amendments. Speak to the real cost of each.]
- **Why it's a strong AI-agent fit:** high query volume, answers that live in a fixed document (perfect for RAG), a clear correctness bar, and a workflow where seconds matter and the human is often on the phone — i.e. voice-native.
- **The commercial size of the problem:** [Sketch the value. Screening failure and amendment costs are publicly documented — cite a public figure for per-patient screening cost or per-amendment cost rather than anything internal. Even a rough, sourced order-of-magnitude makes the business case land.]

---

## 3. The use case I chose (and what I deliberately didn't)

*Deployment strategists are trusted for judgment about scope. Show it.*

- **Chosen:** a read-only, protocol-grounded Q&A agent for site staff.
- **Deliberately excluded from v1:** [e.g. patient-facing enrollment, eligibility *adjudication* for named patients, EDC write-back. Explain why each is out of scope for a first deployment — regulatory risk, trust boundary, change-management load.]
- **The trust boundary that defines the design:** the agent explains what the protocol *says*; it never rules on whether a specific patient qualifies. [Explain why that line is the whole ballgame in a regulated setting.]

---

## 4. What I built

*Be precise about built vs. described. Maturity honesty builds trust.*

**Built (working demo):**
- ElevenAgents agent grounded in a real [public/synthetic] Phase [II] protocol via RAG (embedding model: [multilingual_e5_large_instruct]).
- Production guardrails: PII redaction, `trust_context: low` (external/untrusted participants), out-of-scope refusal, AI self-disclosure.
- A success-evaluation criterion (`answered_from_protocol`) + conversation data collection, giving a real quality-and-adoption analytics view.

**Designed but not built this iteration (with a clear path):**
- **Tool-calling** to a site/EDC system for live patient-context lookups — [describe the webhook/MCP tool you'd add, and the approval-policy scoping it would need given the low trust context].
- **Knowledge-base-as-code:** a CI/CD pipeline where a protocol amendment triggers an API call to replace the knowledge base document and recompute the RAG index, so every agent instantly reflects the current protocol version. [Tie this to treating documentation like code.]

*(Insert 2–3 screenshots: a grounded answer with a citation, the guardrail refusing a patient-specific interpretation, and the analytics/evaluation view.)*

---

## 5. Architecture at a glance

[A simple 4–5 box diagram or bulleted flow:]
Caller (voice/web widget) → ElevenAgents (turn-taking + LLM) → RAG retrieval over indexed protocol → grounded response, with guardrails + PII redaction wrapping the loop and post-call evaluation feeding analytics. [Note where the future tool-calling and KB-as-code pipeline plug in.]

---

## 6. Production-readiness (the part most demos skip)

*This section directly answers "can this person take an agent to production," which is the core of the role.*

- **Regulatory / privacy:** HIPAA considerations, PII redaction, audio-retention settings, AI disclosure. [What you'd need for a real pharma deployment — BAA, zero-retention mode, audit logging.]
- **Trust & safety:** why `low` trust context, scoped tool access, and the patient-interpretation guardrail exist by design.
- **Quality measurement:** the success-evaluation criterion as an ongoing quality gate; how you'd track it across thousands of conversations and catch drift.
- **Known gaps:** [e.g. production monitoring depth, human-in-the-loop for edge cases. Naming real limitations is a strength here.]

---

## 7. Rollout & adoption plan

*Practice the GTM / delivery muscle the role wants and your background is lighter on.*

- **Pilot:** [one therapeutic area / a handful of sites, N weeks, with a specific success metric — e.g. reduction in medical-monitor query volume, or time-to-answer.]
- **Expansion:** [multi-protocol, multi-language (70+ supported), telephony for sites without app access.]
- **Change management:** [how you'd drive site-staff adoption — the hard part; tie to your real experience driving firm-wide adoption of a tool.]
- **How I'd measure success:** [the 2–3 numbers a sponsor would actually care about.]

---

## 8. A commercial model (deliberately stretching my range)

*You don't need this to be perfect — you need to show you'll engage with it. Flag it as a first-pass.*

- **Value created:** [tie to Section 2's cost figures — e.g. X% fewer screening failures across Y patients = $Z.]
- **Pricing options to explore with a GTM partner:**
  - Platform/license fee per site or per study.
  - Usage-based (per conversation-minute).
  - **Outcome-based:** a share of measured savings (e.g. reduced amendments / faster recruitment) — [note why outcome-based is attractive but needs a clean baseline and attribution, which the analytics layer helps establish.]
- **What I'd want to learn from the customer before pricing:** [baseline metrics, procurement model, budget owner.]

---

## 9. What field insight this surfaced

*The role feeds insights back to Research/product. Show you think that way.*

[1–2 honest observations from building — e.g. where RAG retrieval struggled on tabular protocol data like visit-schedule grids, or where expressive voice mattered for trust with clinical staff. A genuine product-improvement idea here is gold.]

---

## Demo

- **Live agent:** [link]
- **3-min walkthrough:** [Loom link]
- **Code:** [repo link]

---

### A note on scope and honesty (keep this ethos, maybe not the heading)

Everything here uses public or synthetic protocol data — no client or proprietary material. I've been explicit about what's built versus designed, because in a deployment role the trust is the product.
