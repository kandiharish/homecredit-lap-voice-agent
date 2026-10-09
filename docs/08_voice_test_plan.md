# Phase 7 — Voice Test Plan (Submission Recordings)

Prompt version: **v3** · Retell → **Test Audio** (web call) · Agent: Priya (female) · Customer: Rahul Sharma

**Budget:** ~$0.13/min × ~2.5 min × 14 calls ≈ $5 of the $10 free credit.

**Language setup:** English calls use the English agent set to **English (India) only**; Hinglish-heavy calls belong to the Hindi agent (Hindi (India) only). See test log P1/P2.

## How to do every call
1. **Before each session:** update `current_time` in Default Dynamic Variables to your real time.
2. Click **Test Audio** → **Run Test** → speak.
3. **Talk like a real customer**: fillers ("umm", "actually", "matlab"), full sentences, natural pace. The lines below are a guide — say them in your own way.
4. After the call: **Call History** → open the call → download **recording** + copy **transcript** → check the **post-call extraction**.
5. Save as `V01_happy_path_QUALIFIED.mp3` / `.txt` etc. (names below).
6. If a call goes wrong because of *your* mistake (you misspoke), just redo it. If the *agent* is wrong, keep it and tell me — that's a bug to fix.

---

## English calls (agent language = English)

### V01 — Happy path, natural speech → QUALIFIED
File: `V01_happy_path_QUALIFIED`
- "Yes, speaking." → "Yeah sure, go ahead." → "Okay."
- "Umm, it's a flat, a 2BHK in Pune."
- "It's only in my name."
- "Yes, the original registry is at home."
- "Around forty lakh."
- "I'm salaried, salary comes directly in my bank account."
- "Maybe around eighty, eighty-five lakh."
- "Ten years should be fine."
**Expect:** all 7 asked once each, QUALIFIED, call ends.

### V02 — Showcase: out-of-order + diversion + over cap → QUALIFIED ⭐
File: `V02_out_of_order_diversion_cap_QUALIFIED`
- "Yes, speaking." → "Okay, but quickly please." → "Okay."
- "So it's a residential flat, jointly owned with my wife, worth around one point two crore. By the way, what's the interest rate?"
- "Yes, the original registry is at home."
- "Actually I need one crore."
- "Okay fine, seventy-five is fine."
- "I have my own business, my payments come on GPay."
- "Sixty months."
**Expect:** no rate quoted, never re-asks ownership or value, explains 75L cap, 60 months = 5 years, QUALIFIED.

### V03 — Agricultural → DISQUALIFIED
File: `V03_agricultural_DISQUALIFIED`
- Confirm identity, available, okay.
- "It's agricultural land, in my village."
**Expect:** immediate polite disqualification, no more questions.

### V04 — Cash income → clarify → DISQUALIFIED
File: `V04_cash_income_DISQUALIFIED`
- … "My own shop, in my name." → "Yes, originals are there." → "Twenty lakh."
- "I have a kirana store, customers pay cash."
- "No, I keep the cash at home only."
**Expect:** one clarification about deposit, then disqualification.

### V05 — Only photocopies → DISQUALIFIED
File: `V05_no_original_documents_DISQUALIFIED`
- … "It's my house, sole owner."
- "Umm, I only have photocopies, the originals got lost when we shifted."
**Expect:** disqualification.

### V06 — Tenure too long → DISQUALIFIED
File: `V06_tenure_20_years_DISQUALIFIED`
- Answer items 1–6 as eligible (any order).
- "I'd like twenty years."
**Expect:** disqualification. Must NOT suggest "would fifteen work?" before you answered.

### V07 — Declines the 75L cap → DISQUALIFIED
File: `V07_declines_75L_cap_DISQUALIFIED`
- … "I need ninety lakh."
- "No, seventy-five won't work for me, I need the full ninety."
**Expect:** cap explained once, then polite close.

### V08 — Existing loan / EMI reduction → TRANSFER
File: `V08_existing_loan_TRANSFER`
- … "It's my flat. But actually there's already a home loan running on it and the EMI is too high, I want to reduce it."
**Expect:** transfer-specialist message, call ends, no eligibility questions.

### V09 — Busy → CALLBACK
File: `V09_busy_CALLBACK`
- "Yes, speaking." → "I'm driving right now, call me later."
- "Tomorrow evening, after six."
**Expect:** no pitch, callback time confirmed with the correct day, call ends.

### V10 — Family member answers → no offer details
File: `V10_family_member_CALLBACK`
- "Rahul? He's my son, he's not at home. Who is this?"
- "He'll be back by seven in the evening."
**Expect:** introduces self, **never mentions loan details**, asks/accepts a good time, call ends.

### V11 — Interruptions + RAG question + car loan → QUALIFIED
File: `V11_interruptions_rag_QUALIFIED`
- **Interrupt** the agent mid-sentence during the offer: "Wait wait — what documents will I need?" *(answer should come from the sample RAG text)*
- … during the checklist: "I have a car loan, is that a problem?"
- "Am I eligible so far?"
- Complete the rest as eligible.
**Expect:** stops talking when interrupted, answers from RAG, car loan ≠ transfer, "so far so good", QUALIFIED.

### V12 — Not interested → polite close
File: `V12_not_interested`
- "Yes speaking." → "Okay." → after offer: "No, I'm not interested in any loan right now."
**Expect:** no persuasion, thanks, call ends.

---

## Hindi calls (duplicate agent, language_to_speak = Hindi, Hindi voice)

### V13 — Hindi happy path with self-correction → QUALIFIED
File: `V13_hindi_happy_path_QUALIFIED`
- "हाँ जी, बोल रहा हूँ।" → "हाँ, बताइए।" → "ठीक है।"
- "मेरा अपना घर है, सिर्फ मेरे नाम पर।"
- "हाँ, रजिस्ट्री घर पे है।"
- "पचास लाख चाहिए।"
- "नौकरी करता हूँ, सैलरी बैंक में आती है।"
- "करीब एक करोड़ का होगा।"
- "दस साल... नहीं, बारह साल कर दीजिए।"
**Expect:** Hindi in Devanagari voice, **female forms** ("बोल रही हूँ"), 12 years used, QUALIFIED.

### V14 — Hindi transfer
File: `V14_hindi_EMI_TRANSFER`
- … "दुकान है मेरी। पर उस पर पहले से लोन चल रहा है, EMI कम करवानी है।"
**Expect:** Hindi transfer message, call ends.

---

## Results table (final set)

| ID | Scenario | Prompt | Expected | Actual | Pass | Files |
|----|----------|--------|----------|--------|------|-------|
| V01 | Happy path | v3 | QUALIFIED | QUALIFIED, all 7 asked once | ✅ | V01_happy_path_QUALIFIED |
| V02 | Out-of-order + rate + 1 crore + GPay + 60 months (Hinglish) | v4 | QUALIFIED | 3 facts captured, rate deflected, 75L cap, value not re-asked, QUALIFIED | ✅ | V02_out_of_order_diversion_cap_QUALIFIED |
| V03 | Agricultural (Hinglish) | v4 | DISQUALIFIED | Immediate DQ | ✅ | V03_agricultural_DISQUALIFIED (+ V03b on v5) |
| V04 | Kirana cash, not deposited (Hinglish) | v4 | Clarify → DQ | Clarified deposit → DQ | ✅ | V04_cash_income_DISQUALIFIED (+ V04b on v5, also shows "bees lakh" mis-hearing recovered) |
| V05 | Only photocopies | v5 | DISQUALIFIED | DQ | ✅ | V05_no_original_documents_DISQUALIFIED |
| V06 | Tenure 20 years | v5 | DISQUALIFIED, no coaching | DQ, range not volunteered | ✅ | V06_tenure_20_years_DISQUALIFIED |
| V07 | 90 lakh → accepts 75 → later reverses to "need full ninety" | v5 | DISQUALIFIED | Latest answer wins → DQ | ✅ | V07_declines_75L_cap_DISQUALIFIED |
| V08 | Existing home loan + reduce EMI (Hinglish) | v5 | TRANSFER | EMI-reason transfer message | ✅ | V08_existing_loan_EMI_TRANSFER |
| V09 | Busy, "tomorrow nine PM" | v5 | CALLBACK | No pitch; "Saturday at nine in the evening" | ✅ | V09_busy_CALLBACK |
| V10 | Family member asks "is it important?" | v5 | CALLBACK, no details, call ends | No details shared ✅, but goodbye said twice and line left open | ❌ B7 (fixed in v6) → re-record on v6 | dev07 |
| V11 | Interrupts during offer: "what documents?" (RAG) → originals lost | v5 | Stops, answers from RAG | Stopped, answered from RAG, DQ on lost originals | ✅ | V11_interruption_rag_question_DISQUALIFIED |
| V12 | Annoyed → not interested | — | Explain, then polite close | Not recorded | ⏳ | — |
| V13 | Hindi call (language_to_speak = Hindi) | — | QUALIFIED in Hindi, female forms | Not yet — the Hindi attempt was made with language_to_speak = English (agent stayed English → F3, fixed in v6) | ⏳ re-record on v6 | dev08 |
| V14 | Hindi transfer | — | TRANSFER in Hindi | Not recorded | ⏳ optional | — |

Development recordings that exposed issues are in `recordings/dev/` (dev01–dev08), each named after the bug or feedback item it revealed.
