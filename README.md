# Home Credit LAP Qualification — AI Voice Agent

**SalesAgents AI · AI Intern (Prompt Engineer) assignment**
Voice agent that calls existing Home Credit customers, presents a pre-approved Loan Against Property (LAP) offer of up to ₹75 lakh, runs a 7-point preliminary eligibility check, and routes every call to the right outcome.

| Deliverable | Where |
|---|---|
| **Final system prompt** | [`SYSTEM_PROMPT.md`](SYSTEM_PROMPT.md) (version 7) |
| **Call recordings + transcripts** | Published page (audio players + transcripts) · [`recordings/`](recordings/) · [`transcripts/`](transcripts/) |
| **Test results** | [Results table](#5-test-results) below · full log in [`docs/07_test_log.md`](docs/07_test_log.md) |
| **Approach & design** | This document + [`docs/`](docs/) |
| Platform | Retell AI — single-prompt agent, GPT 5.6 Terra, ElevenLabs voice "Monika" (Indian English), speech recognition English (India) + Hindi (India) |

---

## 1. How I approached the problem

I did not start by writing the prompt. I worked in phases, each producing a document that the next phase builds on:

| Phase | Output | Why |
|---|---|---|
| 1. Requirements → rules | [`01_requirements_and_rules.md`](docs/01_requirements_and_rules.md) — 17 traceable requirements, an eligibility rule table, and **39 design decisions** for cases the brief leaves open | A prompt can only encode decisions you have made. Undecided cases are where LLM behaviour becomes inconsistent. |
| 2. How real customers answer | [`02_real_customer_answer_bank.md`](docs/02_real_customer_answer_bank.md) — ~150 realistic Indian customer replies (Hinglish, local terms, vague and multi-part answers) with the correct interpretation | Real customers say "registry ghar pe hai", "gaon mein ghar hai", "payment GPay pe aata hai" — not "Residential / Yes / Bank". |
| 3. State & flow design | [`03_state_and_flow_design.md`](docs/03_state_and_flow_design.md) — state schema, per-turn algorithm, exit priorities, flowchart | Out-of-order answers, diversions and immediate exits are a state problem, not a wording problem. |
| 4. Conversation design | [`04_conversation_scripts.md`](docs/04_conversation_scripts.md) — every spoken line in English and Hindi, gendered Hindi forms, closings | Voice needs short, speakable lines; Hindi grammar changes with the agent's gender. |
| 5. System prompt | [`SYSTEM_PROMPT.md`](SYSTEM_PROMPT.md) | Compiles phases 1–4 into one instruction document. |
| 6. Test → fix → re-test | 9 text tests + 22 voice calls, 8 bugs found and fixed across 7 prompt versions ([`prompt_versions/`](prompt_versions/)) | The first prompt is a hypothesis; testing with realistic speech is what made it reliable. |

## 2. Core design

### The per-turn algorithm (the heart of the prompt)
On every customer turn the agent silently runs:

```
1. EXTRACT   every fact in the reply — any checklist item, any order, several at once
2. EXITS     first match wins:
             do-not-call > not the customer > busy > TRANSFER > DISQUALIFY > not interested
3. RESOLVE   pending items (the 75-lakh cap question, one clarification per item)
4. ANSWER    the customer's question in one sentence (prompt or RAG facts only)
5. ADVANCE   check items 1→7 against the WHOLE conversation; ask the FIRST incomplete one
```

Step 5 — "always ask the first incomplete item, checked against the whole conversation" — satisfies three requirements with one rule: sequential capture, never re-asking out-of-order answers, and returning to items skipped during a diversion. The QUALIFIED hand-off is gated on all seven items being complete and passed.

### Memory
LLMs are stateless; the platform re-sends the full transcript every turn. The transcript is therefore the agent's memory, and the prompt makes the agent rebuild the checklist from it before every reply. The agent also confirms multi-fact answers back in plain words ("Got it — a jointly owned flat, worth around one crore"), which writes clean facts into the transcript and makes them easy to find later. I chose not to add an external state store: it adds latency on every turn and can go out of sync when a customer corrects themselves.

### Prompt structure
Critical rules at the top → call context (all 11 variables) → language, gender & voice style → call flow → the 7 items with PASS / FAIL / CLARIFY rules and real customer vocabulary → per-turn procedure → transfer detection (including what is *not* a transfer) → diversions → hearing problems → closings in EN + HI → 14 short examples → a final self-check list. The critical rules appear at both the start and the end of the prompt.

### All 11 required variables
`company_name`, `customer_name`, `agent_name`, `agent_gender`, `current_date`, `current_day`, `current_time`, `additional_context_from_rag`, `language_to_speak`, `conversation_history`, `customer_utterance`. If a variable arrives empty or unfilled, the prompt tells the agent to fall back on the live conversation.

## 3. Key design decisions (selected)

| Situation not specified in the brief | Decision | Reason |
|---|---|---|
| A reply contains both a disqualifier and a transfer signal | **Transfer wins** | An existing-loan customer is not on the fresh-loan path; don't discard a lead. |
| "Original papers are with the bank" | **Transfer** | Banks hold title deeds while a loan is running on the property. |
| "Customers pay me in cash" | **Clarify** whether it is deposited; deposited cash = bank income | The rule exists for verifiable income. Rejecting every shop owner who banks daily cash would remove the core self-employed segment. |
| Car loan / home loan on another property / "keep the new EMI low" | **Not** a transfer | The brief's trigger is a loan *on this property* or reducing the *current* EMI. |
| "Gaon mein ghar hai" | Residential | A village house is not agricultural land. |
| Market value | Captured, **never** a reason to fail | The brief sets no threshold; inventing one would be wrong. |
| Customer asks "is twenty years okay?" | Answer honestly (3–15 years) | A question is not a failing answer. The agent only avoids *volunteering* criteria before the customer answers. |
| Disqualification | Polite close **with one short reason** | More transparent and advisory than a bare rejection; the decision is not reversed. |
| Garbled or one-word reply | Treated as a hearing problem; max 2 attempts, then a polite callback close | **Never disqualify because of bad audio.** |
| Family member answers | No offer details, even if asked "is it important?"; "nothing urgent, a service call" | Privacy in financial services. |
| Customer speaks only Hindi on an English call | Switch EN↔HI on request (or offer once after two Hindi-only turns); Hinglish mixing alone is not a reason to switch | Matches the reference guide's guidance on `language_to_speak`. |

The full list (D1–D39) and the reasoning behind each decision is in [`docs/01_requirements_and_rules.md`](docs/01_requirements_and_rules.md).

## 4. Platform configuration and trade-offs

- **Agent type: single prompt** (not a node-based flow). Out-of-order answers break rigid node graphs; one LLM with the full conversation handles them naturally.
- **Speech recognition: English (India) + Hindi (India).** Tested both ways: English-only dropped the Hindi words in Hinglish replies ("Haan, registry ghar pe hai" arrived as "Register"); the bilingual setting captured them correctly (e.g. "agricultural land है मेरा गांव में"). `language_to_speak` still controls the reply language.
- **Voice:** an early bilingual configuration made the ElevenLabs voice sound uneven. It was fixed through the voice settings, and offer wording was simplified ("see if you qualify" instead of "check your eligibility") because short words are easier for TTS.
- **`current_time`** is a Retell system variable, so I set `system_timezone = Asia/Kolkata`; the prompt receives live IST time.
- **`conversation_history` / `customer_utterance`:** Retell already sends the live transcript every turn, so these are filled with a pointer to the live transcript. In a production pipeline they would carry the actual history and latest utterance.
- **`additional_context_from_rag`:** filled with clearly labelled **sample test data** (documents list, fee, disbursal time) — not real Home Credit terms — to test that the agent answers from RAG and says "our senior loan expert will confirm" when the answer isn't there.
- **Post-call extraction** enabled for a call summary and outcome on every call.

## 5. Test results

Each recording is the full call audio; each transcript is the platform transcript (speech-to-text errors left as-is).

| ID | Scenario | Expected | Result | Prompt |
|---|---|---|---|---|
| V01 | Happy path, natural speech | QUALIFIED | ✅ All 7 asked once, QUALIFIED | v3 |
| V02 | Hinglish; 3 facts in one answer + interest-rate question + asks ₹1 crore + GPay + "sixty months" | QUALIFIED | ✅ Rate deflected, 75L cap offered, market value not re-asked, 60 months = 5 years | v4 |
| V03 | "Agricultural land है मेरा गांव में" | DISQUALIFIED | ✅ Immediate | v4 (V03b: v5) |
| V04 | Kirana store, customers pay cash, not deposited | Clarify → DISQUALIFIED | ✅ One clarification, then DQ (V04b also shows recovery from "bees lakh" heard as "these lakh") | v4 (V04b: v5) |
| V05 | Only photocopies, originals lost | DISQUALIFIED | ✅ | v5 |
| V06 | Tenure 20 years | DISQUALIFIED, no coaching | ✅ | v5 |
| V07 | Asks ₹90 lakh → accepts 75 → later insists on 90 | DISQUALIFIED | ✅ Latest answer wins | v5 |
| V08 | Home loan already running, wants lower EMI (Hinglish) | TRANSFER | ✅ | v5 |
| V09 | Driving, "call me tomorrow at nine PM" | CALLBACK | ✅ No pitch; "Saturday at nine in the evening" | v5 |
| V10 | Family member asks "is it important?" | No details shared, call ends | ⚠️ Voice call on v5 shared no details but left the line open (bug B7, in `recordings/dev/`); fix verified in text testing on v6 | v5 / v6 |
| V11 | Interrupts during the offer: "what documents will I need?" | Stops, answers from RAG | ✅ Stopped mid-sentence, answered from RAG, then DQ on lost originals | v5 |
| V13 | Full call in Hindi, cash answer, self-correction "दस… बारह… तेरह साल" | QUALIFIED in Hindi, female forms | ✅ Female forms throughout ("बोल रही हूँ", "समझ नहीं पाई"), cash clarified → deposited, final correction (13 years) used, qualified. Closing did not trigger end_call (bug B8 — fixed in v7, verified in text testing) | v6 |

## 6. Bugs found and fixed

| Bug | Found by | Root cause | Fix |
|---|---|---|---|
| B1 | Market value re-asked late in the call although given earlier | No per-item "already answered?" check before asking | Explicit check of items 1→7 against the whole conversation + example (v2) |
| B2 | "Customers pay cash" disqualified without asking about deposits | "Disqualify immediately" outweighed "clarify when unclear" | Disqualify only on a *clearly* failing answer; cash answer must be clarified (v3) |
| B3 | Loyalty framing dropped when the customer was in a hurry | Shortening removed it | Loyalty framing made mandatory (v3) |
| B4 | Documents question asked 3 times; risk of disqualifying on bad audio | No hearing-problem handling | Hearing-problem rule: max 2 attempts, never disqualify on audio (v4) |
| B5 | Irritated customer ("why are you calling me?") treated as do-not-call | "Abusive → DNC" too broad | DNC only on explicit request; annoyed customers get an explanation (v5) |
| B6 | "Haan ji, speaking" transcribed as "Ajay speaking" → ended as wrong person | No confirmation before the wrong-person exit | Confirm the name once before ending (v5) |
| B7 | Family member call: goodbye said, line left open | No defined family closing; goodbye ≠ end_call | Family-member closing + "any goodbye = end_call" (v6) |
| B8 | Hindi closings did not trigger end_call | Hindi example lacked the end_call note; tool description English-only | Hindi goodbyes listed explicitly, Hindi examples end with end_call, tool description updated (v7) |

Tester feedback also changed three behaviours in v6: answer questions about limits, give a reason when disqualifying, and switch to Hindi when the customer needs it. Development calls that exposed these issues are kept in [`recordings/dev/`](recordings/dev/) as evidence.

## 7. Limitations and next steps

- Recordings span prompt versions v3–v6 because the prompt kept improving while testing; each is labelled. The final prompt (v7) only adds the Hindi end-of-call fix.
- Testing was done with browser web calls, not a phone line; a telephony call would add carrier audio quality.
- The free trial limited the number of voice calls; a larger suite would run the text-based scenarios as automated simulation tests after every prompt change.
- In production: write the extracted fields (outcome, property type, amount, etc.) to the CRM after each call, pass callback times to a scheduler, and send hand-offs straight to the senior loan expert's queue.

## Repository layout

```
SYSTEM_PROMPT.md        final system prompt (v7)
index.html              published page (audio players + transcripts)
docs/                   requirements, answer bank, state design, scripts, platform setup, test log, voice plan
prompt_versions/        v1 → v7, see changelog in docs/07_test_log.md
recordings/             final call recordings (MP3); dev/ = calls that exposed bugs
transcripts/            matching transcripts; dev/ likewise
```
