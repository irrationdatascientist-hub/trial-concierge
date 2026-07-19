# Trial Concierge — One-Weekend Build Guide

**Goal:** a working ElevenAgents demo — a voice + chat assistant that answers clinical-trial protocol questions (eligibility, visit schedule, endpoints) grounded in a real protocol document via RAG, with production guardrails switched on.

**Scope for this weekend (the deliberate cut):** RAG agent + guardrails + the case-study writeup. Tool-calling and the pricing model are *described*, not built. This keeps the whole strategic narrative intact while fitting two days.

**What you're proving:** that you can locate a high-impact enterprise use case, stand up a grounded agent on the platform, and think about production (HIPAA, trust context, evaluation) — not just wire up a happy-path chatbot.

---

## Before you start (30 min, do this Friday night)

1. **Create an ElevenLabs account** and generate an API key from the workspace settings. (You do this yourself — never paste the key into any chat or shared doc; keep it in an environment variable.)
2. **Set the key locally:**
   ```bash
   export ELEVENLABS_API_KEY="your_key_here"
   ```
3. **Install the SDK:**
   ```bash
   pip install elevenlabs
   ```
4. **Find a public, de-identified sample protocol.** Do NOT use anything from your PwC work — that's client-confidential. Instead grab a publicly published protocol: ClinicalTrials.gov attachments, an NIH/NIAID sample protocol template, or a published trial's supplementary protocol PDF. A generic Phase II oncology or vaccine protocol is perfect. Save it as `protocol.pdf`.

> **Confidentiality guardrail:** everything in this demo must be public or synthetic. Your credibility in the writeup comes from your *domain judgment*, which you can express without any PwC material. Never reference client names, internal metrics you can't source publicly, or proprietary designs.

---

## Day 1 — Working grounded agent (target: 6–8 hours)

### Checkpoint 1: A blank agent talks back (1 hr)

Create an agent via the dashboard (fastest) or API. Dashboard: New assistant → Blank template → set a name like "Trial Concierge." Give it this starter system prompt, then press **Test AI agent** and confirm it responds by voice.

```
You are Trial Concierge, an assistant for clinical research associates and
site coordinators. You answer questions about a specific study protocol —
eligibility criteria, visit schedules, endpoint definitions, and dosing
windows — using ONLY the protocol document in your knowledge base.

Rules:
- Ground every answer in the protocol. If the protocol doesn't cover it,
  say so and advise contacting the study medical monitor.
- Never give medical advice or interpret criteria for a specific named
  patient. You explain what the protocol says, not who qualifies.
- Cite the section/criterion when you can (e.g. "per inclusion criterion 4").
- Keep answers to a couple of sentences unless asked to expand.
```

✅ **Done when:** the agent greets you and holds a basic voice conversation.

### Checkpoint 2: Protocol loaded and RAG-indexed (2–3 hrs)

This is the technical heart of the demo. The flow is: create knowledge base document from file → compute the RAG index → attach it to the agent with RAG enabled.

```python
import os, time, requests

API = "https://api.elevenlabs.io/v1/convai"
H = {"xi-api-key": os.environ["ELEVENLABS_API_KEY"]}

# 1) Create a knowledge base document from the protocol file
with open("protocol.pdf", "rb") as f:
    r = requests.post(
        f"{API}/knowledge-base/file",
        headers=H,
        files={"file": ("protocol.pdf", f, "application/pdf")},
        data={"name": "Study XYZ Protocol v1"},
    )
r.raise_for_status()
doc_id = r.json()["id"]
print("document id:", doc_id)

# 2) Trigger RAG indexing (pick an allowed embedding model)
r = requests.post(
    f"{API}/knowledge-base/{doc_id}/rag-index",
    headers={**H, "Content-Type": "application/json"},
    json={"model": "multilingual_e5_large_instruct"},
)
r.raise_for_status()
print("rag status:", r.json())

# 3) Poll until indexing completes (large docs take a few minutes)
for _ in range(30):
    s = requests.get(f"{API}/knowledge-base/{doc_id}/rag-index", headers=H).json()
    print(s)
    if str(s).lower().find("succeeded") != -1 or str(s).lower().find("complete") != -1:
        break
    time.sleep(10)
```

> **Note on exact field names:** the SDK/endpoint response shapes shift between versions. If a key like `["id"]` or the status string doesn't match, hit the "Try it" panel on the [Compute RAG index](https://elevenlabs.io/docs/eleven-agents/api-reference/knowledge-base/compute-rag-index) and [Create-from-file](https://elevenlabs.io/docs/eleven-agents/api-reference/knowledge-base/create-from-file) docs pages to see the live response, and adjust. Budget time for one or two of these mismatches — that's the normal API-friction tax.

**Then attach the document to the agent with RAG on.** Easiest in the dashboard: agent settings → Knowledge Base → add the document → toggle **Use RAG**. Under Advanced, set the embedding model to match what you indexed with, and leave max chunks / max vector distance at defaults to start. Set the document's retrieval mode to **Auto** (retrieved only when relevant).

✅ **Done when:** you ask "What are the key inclusion criteria?" and the agent answers *from the protocol*, ideally citing a criterion number. Ask something the protocol doesn't cover and confirm it declines gracefully.

### Checkpoint 3: Tune until it's trustworthy (2 hrs)

This is where the fiddly time goes and where a strategist earns the demo. Run ~10 realistic questions a CRA would actually ask:

- "What's the washout period before randomization?"
- "Can a patient on metformin be enrolled?" (should explain the criterion, NOT rule on the patient)
- "How many days is the screening window?"
- "What's the primary endpoint and how is it measured?"
- "What do I do if a patient misses the Day 28 visit?" (tests the graceful-fallback path)

Tighten the system prompt each time it over-reaches, hallucinates, or fails to cite. The failure you most want to eliminate: the agent *interpreting eligibility for a specific patient*. That's the line between a helpful protocol reference and a regulatory problem — and naming that line in your writeup is exactly the kind of judgment the role screens for.

✅ **Done when:** all 10 questions produce grounded, appropriately-bounded answers.

---

## Day 2 — Guardrails, polish, and the writeup (target: 6–8 hours)

### Checkpoint 4: Production guardrails (1–2 hrs)

Turn on the things that make this an *enterprise* demo rather than a hackathon toy. In agent settings:

- **PII redaction** — on. Protocol Q&A shouldn't capture patient identifiers; if a caller says a name, it's redacted from logs.
- **Trust context: `low`** — this agent serves external site staff (untrusted participants), so outputs should be vetted and tool access scoped. Setting this deliberately, and explaining *why* in the writeup, is a strong signal.
- **Out-of-scope / off-topic guardrails** — keep it on the protocol; refuse medical-advice and unrelated requests.
- **Disclosure** — the agent should identify itself as AI at the start (also a legal requirement in many jurisdictions for voice agents).

### Checkpoint 5: Evaluation criteria + data collection (1 hr)

Under the agent's **Analysis** tab, add a success-evaluation criterion — this is your "how I'd measure adoption and quality" story made concrete:

- **Name:** `answered_from_protocol`
- **Prompt:** "The assistant answered the user's protocol question using the knowledge base, cited the relevant section where possible, and did NOT interpret eligibility for a specific named patient."

Add a data-collection item (`user_question`, string) that extracts each caller's question, so you can show a real "here's what site staff actually ask" analytics view after a few test conversations.

✅ **Done when:** you've run 5+ test conversations and can see pass/fail evaluations and collected questions in the Call History tab. Screenshot this — it's a money slide for the writeup.

### Checkpoint 6: Make it demoable (1 hr)

Embed the widget so you have a clean link/screen to record:

```html
<elevenlabs-convai agent-id="YOUR_AGENT_ID"></elevenlabs-convai>
<script src="https://unpkg.com/@elevenlabs/convai-widget-embed" async type="text/javascript"></script>
```

Drop that into a bare HTML page (or any no-code host). Record a **3-minute Loom**: state the use case in one line, ask 3–4 sharp questions live, deliberately trigger the "I can't interpret for a specific patient" guardrail, and show the analytics tab. That guardrail moment is the single most impressive 15 seconds — it proves you built for production, not applause.

### Checkpoint 7: Write the case study (3–4 hrs)

Use the separate scaffold document. This is what actually gets you the role — spend real time here.

---

## If you fall behind

Cut in this order (protect the narrative, drop the polish):
1. Skip the widget embed — record the Loom straight from the dashboard's Test panel.
2. Drop data-collection; keep the one success-evaluation criterion.
3. If RAG indexing fights you past ~3 hrs, fall back to pasting key protocol sections into the system prompt as a stopgap, note in the writeup that production would use RAG (which you understand and can describe), and move on. A shipped narrow demo beats a half-built broad one.

## What NOT to do

- Don't add voice cloning, telephony, or multi-channel. Scope creep is the enemy; the role rewards a tight demo + sharp thinking.
- Don't use any PwC / client material. Public or synthetic only.
- Don't oversell. In the writeup, be explicit about what's built vs. described — deployment strategists are trusted precisely because they're straight about maturity.
