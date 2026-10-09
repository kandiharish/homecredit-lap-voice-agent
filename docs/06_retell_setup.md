# Phase 5 — Retell AI Setup Guide

Labels verified against Retell's documentation (docs.retellai.com). If something differs slightly on screen, use the closest match.

**Cost:** new accounts get **$10 free credits, no card needed**. Browser test calls need no phone number. The per-minute price is shown next to each model — 15 test calls of ~3 minutes each fit comfortably in the free credits.

---

## Step 1 — Sign up
1. Open **https://dashboard.retellai.com** → sign up (Google sign-in is fine).

## Step 2 — Create the agent
1. Go to the **Agents** tab → **Create an agent**.
2. Choose **Single prompt**.
3. Choose **Build from scratch** (not a template, not "Generate from prompt" — we have our own prompt).
4. Rename the agent: `HomeCredit_LAP_Qualifier_EN_v1`.

## Step 3 — Model (toolbar, top of the agent page)
1. Open the **model dropdown**.
2. Pick a model from **"Versatile and highly intelligent"**, preferably one marked **Suggested**. Do NOT pick from "Fast and cost-efficient" (small models skip rules in long prompts) or "Speech to speech" (no text playground testing).
3. Used: **GPT 5.6 Terra** (Suggested). Its ⚙ settings show only **Structured Output** (leave ON) — this model has **no temperature setting**, so there is nothing to change.

## Step 4 — Voice (toolbar, next to the model)
1. Click the voice control → **Select Voice** dialog.
2. Filters: **Gender = Female**, **Accent = Indian** (if available).
3. Play 2–3 samples, choose the calmest, clearest professional voice. Note its name.
4. Voice speed: keep **1.0x**.

## Step 5 — Language
1. Open the **language picker**.
2. English agent: **English (India)** if listed; otherwise English.
3. Retell automatically tells the agent to respond in this language — this works together with our `{{language_to_speak}}` rule.

## Step 6 — Prompt
1. Open `05_system_prompt_v1.md`, select all, copy.
2. Paste into the large **prompt box** in the middle of the page.

## Step 7 — Welcome Message (below the prompt)
1. Set **AI speaks first** → **Dynamic message** (so the greeting follows our time-of-day rule).
2. **Pause Before Speaking**: about **1 second** (so the agent doesn't speak over "hello").

## Step 8 — End Call tool
1. In the **tools/functions** section click **+ Add** → **End Call**.
2. Name: `end_call`. Description: *"End the call immediately after speaking any closing line (qualified, disqualified, transfer, callback, not interested, wrong person, do not call, no response)."*
3. Save.

## Step 9 — Agent Handbook (beside the prompt)
1. **Personality & Tone → Default Tone: Professional** (NOT "Professional + Conversational"). Retell itself recommends Professional when you have written a detailed personality into your own prompt; Conversational Mode rephrases replies and could alter our closing lines.
2. Leave extra add-ons (filler words, high empathy) **off** for v1 — test our prompt as written first.
3. If **Speech Normalization** is available, turn it **on** (helps numbers and amounts sound natural).

## Step 10 — Speech settings
| Setting | Value | Why |
|---|---|---|
| Background sound | Call center (optional) | Realistic feel |
| Response wait time | Small (~0.5 s) **or** tick "Dynamically adjust" | Callers pause mid-sentence ("woh… kya hai…") |
| Interruption sensitivity | Slightly below default | So "umm" and background noise don't cut the agent off |
| Backchanneling | On | Natural "mm-hmm" while the customer talks |
| Reminder frequency | ~10 s, max 2 | Matches our NO RESPONSE rule |

## Step 11 — Call settings
| Setting | Value |
|---|---|
| End call on silence | 30 s |
| Call duration (max) | 10 min |

## Step 12 — Default Dynamic Variables (Security & fallback settings)
Add these keys and values:

| Variable | Value |
|---|---|
| company_name | Home Credit |
| customer_name | Rahul Sharma |
| agent_name | Priya |
| agent_gender | female |
| current_date | 9 October 2026 *(update on the day you test)* |
| current_day | Friday *(update on the day you test)* |
| system_timezone | Asia/Kolkata *(makes Retell's built-in `{{current_time}}` return live Indian time)* |
| language_to_speak | English |
| additional_context_from_rag | see box below |
| conversation_history | See the live conversation transcript. |
| customer_utterance | See the customer's latest message in the live conversation. |

**Sample RAG text — clearly TEST DATA, not real Home Credit terms:**
```
TEST PRODUCT KNOWLEDGE (sample for testing only): Documents usually needed are KYC (PAN and Aadhaar), last six months of bank statements, income proof, and the original property title documents. Processing fee is up to one percent of the loan amount. Disbursal is usually within seven working days after final approval.
```

ℹ️ **`current_time` is a Retell system variable** — a custom default with that name is not saved. Setting `system_timezone = Asia/Kolkata` makes the built-in `{{current_time}}` return the live IST time (e.g. "Friday, October 9, 2026 at 2:30 PM IST"), so the prompt's `{{current_time}}` variable is always correct without manual updates. Add each default variable one at a time and re-open the panel to confirm it saved.

## Step 13 — Post call extraction
Add these fields (types: selector = fixed choices, text = free text):

| Field | Type | Choices / description |
|---|---|---|
| call_outcome | Selector | QUALIFIED, DISQUALIFIED, TRANSFER, CALLBACK, NOT_INTERESTED, WRONG_PERSON, DO_NOT_CALL, NO_RESPONSE |
| dq_reason | Text | Which eligibility rule failed, if disqualified |
| property_type | Text | Residential / Commercial / Industrial / Mixed / Agricultural |
| ownership | Text | Sole / Joint / Not owner |
| original_documents | Text | Available / Not available / With bank |
| loan_amount | Text | Requested amount in lakh |
| occupation | Text | Salaried / Self-employed / Other |
| income_mode | Text | Bank / Cash |
| market_value | Text | Customer's estimate, or "unsure" |
| tenure_years | Text | Requested repayment years |
| callback_time | Text | Callback time requested, if any |

## Step 14 — First test (text, cheap): LLM Playground
1. Click **Test** → **Test LLM** tab (text chat).
2. Type as the customer: `haan ji, speaking` → `yes go ahead` → `sure` → `it's my flat, only in my name` → `yes originals are at home` → `40 lakh` → `salaried, salary comes in bank` → `around 80 lakh` → `10 years`.
3. Check:
   - [ ] Greeting uses Rahul Sharma, Priya, Home Credit, correct time of day
   - [ ] One question per turn, never re-asks ownership
   - [ ] No bracket notes, labels or `{{...}}` text in replies
   - [ ] QUALIFIED closing, then the **end_call** tool fires
4. Copy the conversation and send it to me.

## Step 15 — First voice test (web call)
1. **Test** → **Test Audio** → allow microphone → talk naturally (same answers, with "umm"s).
2. Listen for: pronunciation of "lakh", amounts said as words, natural pace, no talking over you.
3. Open **Call History** → confirm the recording, transcript and post-call extraction appear.

---

## Final configuration record (fill in — goes into the approach doc)

| Setting | Value used |
|---|---|
| Platform | Retell AI |
| Agent type | Single prompt |
| LLM model | |
| Temperature | |
| Voice (English) | |
| Voice (Hindi) | |
| Language | |
| Default tone | Professional |
| Interruption sensitivity | |
| End-call tool | end_call |
| Prompt version | v1 |
