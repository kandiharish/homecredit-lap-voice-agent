# Phase 6 — Test Log

Platform: Retell AI · Model: GPT 5.6 Terra

Legend: ✅ pass · ⚠️ minor issue · ❌ fail

---

## Summary

| ID | Scenario | Prompt | Mode | Expected | Actual | Result |
|----|----------|--------|------|----------|--------|--------|
| T01 | Happy path, eligible customer | v1 | Text | QUALIFIED + end_call | QUALIFIED + end_call | ✅ (minor ⚠️) |
| T02 | Multi-answer + interest-rate diversion + 1 crore request + GPay + 60 months | v1 | Text | Skip items 2 & 6, deflect rate, offer 75L cap, QUALIFIED | Re-asked market value (item 6) late in the call | ❌ **Bug B1** |
| T03 | Hidden transfer: "original papers bank mein jama hain" | v1 | Text | TRANSFER + end_call | TRANSFER + end_call | ✅ |
| T02-r | Re-run of T02 (regression for B1) | v2 | Text | Skip item 6, ask tenure | Skipped item 6, asked tenure, QUALIFIED + end_call | ✅ B1 fixed (minor ⚠️ B3) |
| T04 | Car loan (not transfer) → "customers cash dete hain" → "cash ghar pe rakhta hoon" | v2 | Text | Continue after car loan; clarify cash once; then DISQUALIFIED | Continued after car loan ✅, but disqualified immediately on "customers cash dete hain" without clarifying | ❌ **Bug B2** |
| T05 | Busy: driving → "kal shaam 6 baje ke baad" | v2 | Text | No pitch; callback confirmed for Saturday evening; end_call | Exactly as expected | ✅ |
| T06 | Contradiction: "residential... actually no, agricultural land" | v2 | Text | Latest answer wins → DISQUALIFIED | DISQUALIFIED + end_call | ✅ |
| T04-r | Re-run of T04: cash, NOT deposited | v3 | Text | Clarify deposit → DISQUALIFIED | Reported pass by tester (transcript to be added) | ✅ B2 fixed |
| T04b | Cash, deposited daily ("roz bank mein jama karta hoon") | v3 | Text | Clarify → PASS (bank) → continue → QUALIFIED | Clarified, accepted as bank, continued, QUALIFIED + end_call | ✅ B2 fixed |

---

## T01 — Happy path (v1, text) ✅

| Check | Result | Note |
|-------|--------|------|
| Variables filled (Rahul Sharma, Priya, Home Credit) | ✅ | No raw `{{...}}` |
| Time-of-day greeting | ✅ | "Good afternoon" at ~12 pm |
| Ownership skipped (given with property type) | ✅ | Anchor confirmation used |
| Income mode skipped (given with occupation) | ✅ | |
| One question per turn, order 1→7 | ✅ | |
| QUALIFIED only after all 7, end_call fired | ✅ | |
| Acknowledgements before questions | ⚠️ | Missing on turns 6–9. **Not repeated in T02/T03** (ack used every turn) → intermittent, not a pattern; monitor |
| "Hi Rahul Sharma" (full name) | ⚠️ → won't fix | Using the full name is safe: we can't reliably tell which part is the first name for every Indian name format |

## T02 — Multi-answer, diversion, cap (v1, text) ❌

| Check | Result | Note |
|-------|--------|------|
| Captured 3 facts at once (residential, joint, 1.2 crore) | ✅ | Anchor: "worth around one crore twenty lakh" |
| Interest rate deflected, no number invented | ✅ | Then returned to documents in the same turn ⭐ |
| Hinglish "registry ghar pe hai" understood | ✅ | |
| 1 crore request → explained 75L cap, offered 75L | ✅ | |
| "75 is fine" accepted | ✅ | |
| GPay → bank, business → self-employed | ✅ | Both parts of item 5 captured in one answer |
| **Market value NOT re-asked** | ❌ | **Bug B1:** asked "what would the property be worth" although given in turn 4 |
| "60 months" → five years | ✅ | Correct conversion; tenure 5 → pass |
| QUALIFIED closing | ✅ | |
| end_call fired | ❓ | Not visible in the pasted transcript — to confirm |

**Bug B1 — root cause:** the model captured the value early but, four turns later, reverted to linear order (5 → 6) instead of checking whether item 6 was already answered. v1 told it to "rebuild the checklist" but never required a per-item check before asking, and had no example of skipping an item *late* in the call.

## T03 — Hidden transfer signal (v1, text) ✅

| Check | Result | Note |
|-------|--------|------|
| "shop hai, mere naam pe" → commercial + sole | ✅ | Ownership skipped |
| "papers bank mein jama hain" → TRANSFER (decision D5) | ✅ ⭐ | Customer never said the word "loan" — agent inferred it |
| No further questions, end_call fired | ✅ | |

---

## T02-r — Regression of B1 (v2, text) ✅
| Check | Result | Note |
|-------|--------|------|
| Market value NOT re-asked; went from item 5 straight to tenure | ✅ | **B1 fixed** |
| end_call fired | ✅ | |
| Loyalty framing in offer | ⚠️ **B3** | Customer said "jaldi"; agent shortened the offer and dropped "As one of our valued customers" — the PDF (§2B) requires presenting it as a loyalty reward |
| Anchor confirmation of 3 facts | ⚠️ | Skipped this time (went straight to the rate answer) — acceptable; facts still captured |

## T04 — Not-a-transfer + cash income (v2, text) ❌
| Check | Result | Note |
|-------|--------|------|
| "car loan running" → NOT transfer, continued | ✅ ⭐ | Example F worked; "That's fine for this check" |
| "customers cash dete hain" → clarify deposit first (decision D30) | ❌ **Bug B2** | Disqualified immediately. A shop owner who deposits cash daily would be wrongly rejected |
| Acknowledgement before "How much loan amount" | ⚠️ | Missing again (also in T01) — low priority |

**Bug B2 — root cause:** Critical Rule 5 ("the moment an answer fails → disqualify") outweighed the item-level clarify rule. "Customers pay cash" *looks like* the cash-income example in the PDF, and the prompt never said such an answer is ambiguous. Example E also started from an English phrasing, so the Hinglish version wasn't matched.

## T05 — Busy customer (v2, text) ✅
| Check | Result | Note |
|-------|--------|------|
| No pitch to a busy customer | ✅ | |
| "kal shaam 6 baje ke baad" resolved to Saturday evening (today Friday) | ✅ ⭐ | Used current_day correctly |
| Replied in English although customer spoke Hinglish | ✅ | language_to_speak = English |
| end_call fired | ✅ | |

## T06 — Contradiction to agricultural (v2, text) ✅
| Check | Result | Note |
|-------|--------|------|
| Latest answer wins ("actually no, agricultural") | ✅ | |
| "village" did not confuse it — land, not a village house | ✅ | |
| Immediate DISQUALIFIED, no further questions, end_call | ✅ | |

---

## T04b — Deposited cash counts as bank (v3, text) ✅
| Check | Result | Note |
|-------|--------|------|
| "customers cash dete hain" → one clarification, not a disqualification | ✅ | **B2 fixed** |
| "roz bank mein jama karta hoon" → bank, PASS (decision D30) | ✅ | "All right, that works." |
| Car loan still treated as not-a-transfer | ✅ | |
| Loyalty framing present in offer | ✅ | **B3 fixed** |
| QUALIFIED + end_call | ✅ | |

---

# Prompt Changelog

## v1 → v2 (fixes Bug B1)
| Change | Where |
|--------|-------|
| Defined "answered" explicitly: answer appears anywhere earlier — in any customer reply **or in the agent's own earlier confirmation** — no matter how many turns ago | Critical Rule 3 |
| Added a mandatory per-item check in Step 5: for each item 1→7, "has the customer already given this?" before asking; warning that early values are easy to forget late in the call | Per-turn procedure, Step 5 |
| Added Example C2: market value given at the start, skipped when the agent reaches item 6 | Examples |
| Final check now says "even many turns ago, or inside my own earlier confirmation" | Final check |

**Regression plan for v2:** re-run T02 (must now skip item 6), then T01 and T03 to confirm nothing broke.

## v2 → v3 (fixes Bug B2 and B3)
| Change | Where | Fixes |
|--------|-------|-------|
| Disqualify only on a **clearly** failing answer; an answer that could be read either way must be clarified once first | Critical Rule 5, Final check | B2 |
| "Customers pay in cash" / "cash business" alone is explicitly NOT a fail → ask "is that money deposited into your bank account?"; FAIL only when confirmed not deposited | Item 5 | B2 |
| Example E rewritten with the exact Hinglish pattern ("customers cash dete hain") → clarify → then disqualify | Examples | B2 |
| Offer must always include the loyalty / valued-customer framing, even when shortened for a hurried customer | Stage 3 | B3 |

**Regression plan for v3:** re-run T04 (must clarify, then DISQUALIFY), plus a new T04b where the cash *is* deposited (must PASS and continue). Spot-check T02 offer wording.

---

# Voice testing findings

## V02 attempt 1 (v3, voice) ❌ — Bug B4 + platform issue P1
Customer said: "Haan, registry ghar pe hai." Transcribed as: "Register" → then "registry" → "registry".
Agent asked about original documents **3 times** (loop).

**P1 — platform (speech recognition):** language was English (India) only, so Hindi words ("haan", "ghar pe hai") were dropped by speech-to-text. The LLM only received one ambiguous word.
→ Fix: language set to **Multiselect: English (India) + Hindi (India)**; Hinglish words added to Boosted Keywords.

**B4 — prompt:** (a) clarification looped 3× despite the "clarify once" rule; (b) latent risk — the documents rule said "still not a clear yes → FAIL", so a pure audio problem could have disqualified an eligible customer.

## v3 → v4 (fixes Bug B4)
| Change | Where |
|--------|-------|
| New **HEARING PROBLEMS** section: a fragment / unrelated word is a hearing problem, not an answer → "Sorry, I didn't catch that" + a short yes/no version; never ask the same question more than twice; never repeat the identical sentence; after two failed attempts → polite "trouble hearing you, our team will call back" closing + end_call | New section |
| **Never disqualify because of a hearing problem** — only on a clearly understood failing answer | Hearing problems, Item 3 |
| Item 3: FAIL only when the customer clearly says no / not sure they have originals; a bare "registry" / "sale deed" reply → confirm with yes/no (usually means yes) | Item 3 |
| Final check: "Have I already asked this twice?" | Final check |


## V02 attempt 2 (v4, voice, multiselect EN+HI) ⚠️ — platform issue P2
Tester feedback: voice sometimes harsh, faster, "eligibility" pronounced twice, some words compressed.

**P2 — root cause:** Monika is an ElevenLabs voice. With English-only, Retell auto-routes it to ElevenLabs' English model (V01 sounded natural). Adding Hindi (multiselect) routes it to a multilingual model, which Retell's docs note can show "occasional pacing or tone quirks". The P1 fix caused P2.

**Decision (trade-off):** follow Retell's guidance — when the caller's language is known in advance, use single-language agents. `language_to_speak` is known before the call, so:
- English agent → **English (India) only** (natural voice restored; Hindi fragments handled by the v4 hearing-problem rule)
- Hindi agent → **Hindi (India) only** (Hindi STT handles Hinglish well)

**Prompt tweak in v4 (wording only):** offer line uses "see if you qualify" instead of "check your eligibility" (hard word for TTS; Hindi line drops "एलिजिबिलिटी"); added a rule to prefer short, easy-to-pronounce words with commas for natural pauses.

## Voice session review (from Retell CSV export, 8 calls)
| # | Call ID | Prompt / config | Scenario | Result |
|---|---------|-----------------|----------|--------|
| 1 | call_5cbac1a… | v3, EN only | V01 happy path | ✅ QUALIFIED |
| 2 | call_eeddf6e… | v3, EN only | V02 attempt 1 | ❌ B4 clarification loop (fixed in v4) |
| 3 | call_f098d36… | v4 | "Haan ji, speaking" transcribed as "Ajay speaking" | ❌ **B6** ended as WRONG PERSON on the real customer |
| 4 | call_13222db… | v4, EN+HI | V02 out-of-order + rate + 1 crore + GPay + 60 months | ✅ QUALIFIED; item 6 not re-asked; Hindi captured in Devanagari |
| 5 | call_86e7cba… | v4, EN+HI | V03 "agricultural land है मेरा गांव में" | ✅ DISQUALIFIED immediately |
| 6 | call_1cd5634… | v4, EN+HI | V04 kirana, cash, not deposited | ✅ clarified once → DISQUALIFIED |
| 7 | call_aec661d… | v4 | Annoyed: "who are you, why are you calling me?" | ❌ **B5** treated as DO NOT CALL |
| 8 | call_520865e… | v4 | Empty user turns (no audio captured) | ⚠️ hearing-problem closing after 2 attempts — behaved per rule; likely tester mic |

End-to-end latency across calls: ~1.1–1.65 s.

**B5 — root cause:** Step 2a said "asks not to be called again, *or is abusive* → DO NOT CALL". The model classified irritation + "why are you calling me?" as abusive.
**B6 — root cause:** the WRONG PERSON exit had no confirmation step, so one speech-to-text error ended a valid call.

## v4 → v5 (fixes B5, B6)
| Change | Where |
|--------|-------|
| DO NOT CALL only on an explicit request ("don't call me again", "remove my number") or abuse that continues after one apology; irritation / "who are you, why are you calling?" is explicitly NOT do-not-call | Step 2a |
| New diversion rule for annoyed/suspicious customers: apologise, one-sentence explanation, ask if it's a good time | Questions and diversions |
| New Example I (annoyed customer) | Examples |
| Before WRONG PERSON: if the reply sounds like another name or is unclear, confirm the name once (STT can hear "Haan ji" as "Ajay") | Stage 1, Step 2b |

**Recordings policy:** development recordings that exposed bugs are kept in `recordings/dev/` as evidence of the iteration process; final submission recordings are made on the final prompt version.

## Tester feedback on v5 (voice, combined EN+HI agent) → v6
| ID | Observation | Fix in v6 |
|----|-------------|-----------|
| B7 | Family member: agent said "have a good day" but end_call did not fire — line stayed open; "is it important?" had no defined answer | New FAMILY MEMBER closing ("nothing urgent, just a service call… we'll call back") + Example K; Critical Rule 10: any goodbye = closing = end_call in the same turn |
| F1 | "Is sixty years okay?" ended the call — a question about the limit was treated as a failing answer | Questions about a limit are answered honestly ("three to fifteen years — what would you prefer?") + Example J; implausible values (e.g. 60 years) confirmed once (probably months) |
| F2 | "Originals not there" → bare "you don't qualify" felt cold | DISQUALIFIED closing now always states one short neutral reason (EN + HI reason list) |
| F3 | Customer who speaks only Hindi on an English call cannot follow | LANGUAGE SWITCHING: mixing ≠ switch; switch EN↔HI only on explicit request, or after two Hindi-only turns + one offer to switch (per Notion guide §24) + Example L |

**Design decision revised (D3, D21):** the agent still never *volunteers* criteria before an answer, but answers *direct questions* about limits honestly, and always gives the reason when disqualifying. Rationale: transparency and the "advisory" tone required by the PDF; no-coaching only applies to unprompted hints.

## v6 voice check (V10 family member, V13 Hindi) → v7
| ID | Observation | Root cause | Fix in v7 |
|----|-------------|------------|-----------|
| B8 | Hindi calls: agent said "आपका दिन शुभ हो" but end_call did not fire — line stayed open (English closings ended correctly) | (1) The only Hindi example (H) ended with the Hindi closing but, unlike every English example, had no "[Then call end_call]" note — the model imitated it; (2) the end_call tool description listed only English goodbyes | Critical Rule 10 now lists Hindi goodbye phrases and says the language never changes the rule; closing section says Hindi closings end the call exactly like English; Example H now ends with the end_call note; new Example M (Hindi transfer + end_call); final check mentions Hindi; Retell end_call tool description updated to include Hindi closings |
