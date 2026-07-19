# Trial Concierge

### A protocol-grounded voice agent for clinical trial sites — built on ElevenAgents

*A deployment case study.*

---

## 1. Summary

Trial Concierge is a voice and chat assistant that gives clinical research associates and site coordinators instant, protocol-grounded answers on eligibility criteria, visit schedules, and endpoint definitions. It's built to reduce the screening failures and protocol deviations that come from misreading a long, dense protocol document mid-visit — and it's designed from the first line to behave the way a tool in a regulated clinical setting has to behave.

I built the working demo on ElevenAgents. This document covers why I chose the use case, what I built versus what I designed, how I'd take it to production, and a first pass at rollout and commercials.

---

## 2. The workflow and the pain point

Clinical research associates (CRAs) and site coordinators run their work off a study protocol — a document that routinely runs well past a hundred pages. When a question comes up in the middle of a patient visit ("is this patient eligible given their medication?", "how long is the screening window?", "how is the primary endpoint measured?"), the honest answer today is that they stop, hunt through the PDF, or email the medical monitor and wait.

That waiting has a cost. Misread inclusion/exclusion criteria lead to screening failures. Missed or misjudged visit windows become protocol deviations. Persistent ambiguity ends in protocol amendments, which are slow and expensive. It's a workflow with three properties that make it an unusually strong fit for a grounded agent: the answer already exists in a fixed document, the query volume is high and repetitive, and the person asking is often mid-task and would benefit from a voice-native answer.


## 3. The use case I chose — and what I deliberately didn't

I scoped this deliberately, because scope judgment is most of the job.

**Chosen:** a read-only, protocol-grounded Q&A agent for site staff. It explains what the protocol says.


**The boundary that defines the design:** the agent informs the human; it never decides for them. In a regulated setting, an AI adjudicating an individual patient's eligibility is a compliance problem, not a feature. Everything in the build serves that line.

---

## 4. What I built

**Built — working demo:**
- An ElevenAgents agent grounded in a `[VERIFY — public / synthetic]` `[VERIFY — phase & therapeutic area, e.g. "Phase II oncology"]` protocol via retrieval-augmented generation (embedding model: `e5-mistral-7b-instruct`).
- Production guardrails: PII redaction, `trust_context: low` (external / untrusted participants), out-of-scope refusal, and AI self-disclosure at the start of every conversation.
- A success-evaluation criterion (`answered_from_protocol`) plus conversation data collection, giving a real quality-and-usage analytics view rather than an assumption that it works.
