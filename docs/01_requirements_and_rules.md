# Phase 1 — Requirements, Eligibility Rules & Design Decisions

Project: Home Credit — Loan Against Property (LAP) Qualification Voice Agent
Source: "SalesAgents AI Hiring Assignment - AI_voice_agent_prep.pdf"

---

## 1. Requirements (traceable to the assignment)

| ID  | Requirement | Source |
|-----|-------------|--------|
| R1  | Call existing customers of {{company_name}} about a pre-approved LAP offer | §1 |
| R2  | Tone: natural, professional, advisory (not a hard sell, not a questionnaire) | §1 |
| R3  | Verify the right customer before discussing the offer | §2A |
| R4  | If customer is busy: acknowledge, ask preferred callback time, end | §2A |
| R5  | Present offer as a loyalty reward, up to ₹75,00,000 (75 lakh) | §2B |
| R6  | Collect 7 eligibility items **sequentially** | §2C |
| R7  | Info given out of order is stored and **never asked again** | §2C |
| R8  | Any failing answer → polite "not eligible for this specific offer at this time" → end | §3 |
| R9  | Existing loan on property / wants lower EMI → "loan-transfer specialist will contact you" → end | §3 |
| R10 | Loan > 75 lakh → explain limit, offer to proceed with 75 lakh | §3 |
| R11 | All 7 answered and passed → "senior loan expert will call back with exact interest rates" | §3 |
| R12 | Prompt must use all 11 variables (see §4 below) | §4 |
| R13 | Agent gender must be adhered to strictly | §4 |
| R14 | Handle full sentences, interruptions, fillers — not a yes/no bot | §5 |
| R15 | Handoff gate: never hand off until every item is asked and answered; return to skipped items | §5 |
| R16 | Deploy on Retell/Bolna, test multiple realistic scenarios | §6 |
| R17 | Submit: system prompt + public links to recordings and transcripts | §6 |

---

## 2. Eligibility Rule Table (the 7 checklist items, in order)

| # | Item | Eligible | Not eligible → action |
|---|------|----------|------------------------|
| 1 | Property type | Residential (house/flat), Commercial (shop/office), Industrial (factory) | Agricultural → DISQUALIFY |
| 2 | Ownership status | Sole **or** Joint (family/partners) | — (no failing value) |
| 3 | Original documents | Originals available for verification (at home, in locker, with family — all fine) | Only photocopies / originals lost / not available → DISQUALIFY |
| 4 | Loan amount | ≤ ₹75,00,000 (exactly 75 lakh is eligible) | > 75 lakh → explain cap, offer 75 lakh (see D2) |
| 5 | Occupation + income mode | (Salaried **or** Self-employed) **AND** income received in bank | Cash income → DISQUALIFY; other occupation → see D8 |
| 6 | Market value | Any estimate — **capture only, no threshold** | — (no failing value; do NOT invent a minimum) |
| 7 | Tenure | 3 to 15 years inclusive | < 3 or > 15 years → DISQUALIFY |

Item 5 is ONE checklist item containing TWO facts. It is complete only when both occupation and income mode are known.

### Global triggers (active at every turn, not just at a specific step)
- **Transfer trigger:** existing loan on the property, wants to reduce current EMI, wants balance transfer / takeover → transfer message → END.
- **Busy trigger:** busy / driving / in a meeting / "call later" → ask callback time → END.
- **Disqualification trigger:** any answer that fails a rule above, at any point → polite message → END.

---

## 3. Prompt Variables (all must appear in the system prompt)

| Variable | Use |
|----------|-----|
| `{{company_name}}` | Introduction, offer, closings |
| `{{customer_name}}` | Verification and personalisation |
| `{{agent_name}}` | Self-introduction |
| `{{agent_gender}}` | Self-reference + gendered grammar (Hindi: "bol rahi hoon" vs "bol raha hoon") |
| `{{current_date}}` / `{{current_day}}` / `{{current_time}}` | Time-appropriate greeting; interpreting "call me tomorrow / after 6" |
| `{{additional_context_from_rag}}` | ONLY source for product facts beyond this prompt |
| `{{language_to_speak}}` | English or Hindi — the call language |
| `{{conversation_history}}` | Source of truth for what has already been collected |
| `{{customer_utterance}}` | The latest thing to extract facts from |

---

## 4. Ambiguities in the Spec & Design Decisions

The assignment leaves these cases unspecified. Each decision is deliberate and is implemented in the system prompt.

| ID | Situation | Decision | Rationale |
|----|-----------|----------|-----------|
| D1 | Customer says something that both disqualifies AND triggers transfer (e.g. "it's agricultural land and I already have a loan on it") | **Transfer wins.** **[Confirmed]** | An existing-loan customer is not on the fresh-loan path, so fresh-loan criteria don't apply to them; the transfer specialist assesses under their own product rules. Avoids wrongly rejecting a possible lead. |
| D2 | Loan > 75 lakh and customer **declines** the 75-lakh cap | Polite close: offer can't cover the higher amount right now → END. If they hesitate ("let me think"), ask once more clearly; no answer = decline. | Spec only defines the "yes" path. A customer who won't accept the cap doesn't fit this offer. |
| D3 | Tenure outside 3–15 | DISQUALIFY. Do **not** volunteer the range ("would 15 work?"). **[Revised after testing]** If the customer *asks* about the limit, answer honestly; confirm implausible values (e.g. "60 years") once. | Spec gives a counter-offer only for loan amount, not tenure. Coaching would invent a rule. |
| D4 | Tenure given in months ("60 months") or vague ("as long as possible") | Convert months → years. If vague, ask "roughly how many years?" without revealing the allowed range. | Prevents false disqualification and avoids coaching. |
| D5 | "The original documents are with the bank" | Treat as **existing loan → Transfer**. | Banks hold original title deeds when a loan is running on the property. Subtle but realistic signal. |
| D6 | Unclear document answer ("papers are somewhere I think") | Ask ONE clarifying question. Clear yes → eligible; clear no / only copies → disqualify. | Never assume a pass on a mandatory criterion. |
| D7 | Mixed income ("part cash, part bank") | Clarify: "Is your main income credited to your bank account?" Mainly bank → eligible; mainly cash → disqualify. | Spec rule is about income mode; primary mode decides it. |
| D8 | Occupation is neither salaried nor self-employed (retired, homemaker, student, unemployed) | Treat as **not meeting criteria** for this offer → polite close. **[Confirmed]** | Spec lists only Salaried and Self-Employed as eligible, same pattern as property types. |
| D9 | Customer doesn't know market value | Ask once for a rough estimate or range ("approximately?"). A range counts as answered. If still unknown, record "customer unsure" and continue. | No threshold exists, so any honest answer satisfies the item; the expert will value it later. |
| D10 | Someone else answers / wrong number | Do NOT disclose any offer details. Ask a good time to reach {{customer_name}} → END. Wrong number → apologise → END. | Customer privacy in financial services. |
| D11 | Customer asks "who is calling / why?" before confirming identity | Give name + company + "regarding an offer for our existing customers", then re-confirm identity. | Builds trust without disclosing details to an unverified person. |
| D12 | Busy but gives no time | Ask once. If still none: "We'll try you again later" → END. Never pitch to a busy customer. | Respects R4 without looping. |
| D13 | Should the agent proactively ask "do you have an existing loan on the property?" | **No.** Detect transfer intent passively throughout the call (and via D5). | Spec defines exactly 7 items; an extra question changes the flow. |
| D14 | Customer corrects themselves ("10 years — actually 12") | Latest clear answer overwrites the earlier one. | Natural speech (R14). |
| D15 | Customer tries to reverse after being disqualified ("oh, then say bank") | Do not reverse. Stay polite, close the call. | Prevents gaming the rules; matches "end the call". |
| D16 | Not interested / do not call | Acknowledge once, no persuasion, thank them → END. | Advisory tone; unwanted calls hurt the brand. |
| D17 | Interest rate, fees, processing time asked | Use `{{additional_context_from_rag}}` if it has the answer; otherwise "our senior loan expert will share exact details". Never invent numbers. Then return to the next missing item. | R11 says the expert gives the rates. Hallucinated rates are a compliance risk. |
| D18 | "Are you a bot / a real person?" | Answer honestly: an AI assistant calling on behalf of {{company_name}}. | Honesty; disclosure norms. |
| D19 | Loan amount unclear ("whatever I'm eligible for" / "maximum") | "Maximum" → 75 lakh. Unknown → ask for a rough figure. | Needs a concrete value to evaluate rule 4. |
| D20 | Language | Speak only in `{{language_to_speak}}`. Hindi mode uses natural Hinglish (English words for EMI, loan, documents, property, as Indian speakers do). Understand code-mixed replies in either mode. | "Must use" in spec; Hinglish matches how real customers talk. |
| D21 | Disqualification message wording | Short, respectful ("this specific offer at this time") **plus one short neutral reason** — **[Revised after testing]** a bare rejection felt cold. No debating, no reversal. | Spec wording; avoids argument. |
| D22 | Silence / unclear audio | Re-prompt briefly up to 2 times, then politely end with a callback note. Confirm critical numbers if they sound off (fifteen vs fifty). | STT errors are common on phone audio. |

### Decisions added after the real-customer analysis (see 02_real_customer_answer_bank.md)

| ID | Situation | Decision | Rationale |
|----|-----------|----------|-----------|
| D23 | "Gaon mein ghar hai" / ancestral house | Residential → eligible. | A village house is not agricultural land. Prevents a false disqualification. |
| D24 | "Plot" / "zameen" / "farmhouse" | Clarify: residential/commercial plot vs agricultural land. Decide on the answer. | The word alone doesn't say whether it's agricultural. |
| D25 | Customer owns several properties | Ask which property they want to pledge; evaluate only that one. | One agricultural property shouldn't disqualify someone who also owns an eligible flat. |
| D26 | Property entirely in someone else's name (father, spouse) | Ask whether the customer's name is on the papers. Yes → Joint ✅. No → not eligible ❌ (polite close). **[Confirmed]** | You can't pledge a property you don't own; "sole or joint" implies the customer is an owner. |
| D27 | Existing loan NOT on this property (car, personal, home loan on another flat) | NOT a transfer → continue the normal flow. | Spec says "existing loan **on the property**". |
| D28 | "Keep the EMI low" for the new loan | NOT a transfer → treat as a tenure preference and ask for years. | Spec's transfer trigger is reducing the **current** EMI. |
| D29 | Income via UPI / GPay / PhonePe / cheque / NEFT / "online" | Bank → eligible. | All of these settle into a bank account. |
| D30 | Customers pay in cash but it's deposited into the bank regularly | Treat as **Bank** ✅. **[Confirmed]** | The rule exists for verifiable income; regularly deposited cash appears in bank statements. Rejecting it would exclude most shop owners. |
| D31 | Purchase price or area given instead of current value | Ask for today's approximate value. | Market value means current value. |
| D32 | Range answer crossing a limit ("70–80 lakh", "12–20 years") | Amount: explain the cap and offer 75 lakh. Tenure: ask for one preferred number (no coaching). | Ranges fully inside the limits are accepted as they are. |
| D33 | Bare "haan / theek hai" to an either/or question | Not an answer → rephrase as a simple choice. | Prevents storing false values. |
| D34 | Customer refuses the questions ("I'll talk to the expert directly") | Explain once that these quick checks are needed. Still refuses → polite close, **no handoff**. | The handoff gate (R15) cannot be bypassed. |
| D35 | Fraud suspicion / OTP fear | Reassure: we never ask for OTP, PIN, card or bank details. | Trust; Indian customers get frequent scam calls. |
| D36 | "Ek minute, hold karo" | Wait patiently; resume with the pending question. | Natural call behaviour. |
| D37 | "Send details on WhatsApp/SMS" | The agent can't send messages; the senior expert will share details. | Don't promise actions the agent can't perform. |
| D38 | Very small loan amount (e.g. 5 lakh) | Eligible — there is no minimum. | Don't invent rules. |
| D39 | Farmer occupation | Treat as self-employed; income mode decides. | Property type and occupation are separate criteria. |

---

## 5. Out of Scope (the agent must NOT do these)
- Quote interest rates, EMIs, fees, or approve or sanction the loan.
- Invent eligibility rules (e.g., minimum market value, LTV ratio, minimum income, age).
- Collect sensitive data not required (PAN, Aadhaar, bank account numbers, OTPs).
- Continue questioning after a disqualification, transfer, busy, or not-interested ending.
