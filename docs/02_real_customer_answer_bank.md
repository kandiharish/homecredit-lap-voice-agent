# Phase 1b — Real Customer Answer Bank

How ordinary Indian customers actually respond on a loan call, and how the agent must interpret each answer.
Written in romanised Hinglish as it typically reaches the LLM from speech-to-text.
These lines also double as **test scripts** for Phase 6.

Legend: ✅ eligible / store · ❌ disqualify · 🔁 transfer · ⏰ callback · ❓ clarify once · ➡️ handle and return to checklist

---

## A. Greeting & Verification

| Customer says | What it really means | Agent action |
|---|---|---|
| "Haan ji" / "Haan bolo" / "Speaking" / "Yes, tell me" (after "Am I speaking with Rahul?") | Identity confirmed | ✅ verified → ask if it's a good time |
| "Haan bolo" (to a generic hello, name not yet asked) | Not a confirmation | ❓ "Am I speaking with {{customer_name}}?" |
| "Kaun bol raha hai?" / "Who is this?" / "Kahan se?" | Wants caller identity first | Introduce name + company → re-confirm identity (D11) |
| "Rahul? Main uska papa bol raha hoon" / "Woh ghar pe nahi hai" | Family member, not customer | No offer details. Ask good time to reach him → ⏰ END (D10) |
| "Wrong number" / "Yahan koi Rahul nahi hai" | Wrong person | Apologise → END |
| "Home Credit? Maine toh loan close kar diya tha" | Past customer, still the right person | ✅ "Yes, this offer is for our valued customers like you" → continue |
| "Mera number kahan se mila?" | Privacy concern | "You're a registered {{company_name}} customer" → continue |
| "Fraud call toh nahi hai? OTP maangoge?" | Suspicion | Reassure: we never ask for OTP, PIN or bank details on call → continue (D35) |
| "Robot ho kya?" / "Ladki real hai?" | AI question | Honest: AI assistant from {{company_name}} (D18) |

## B. Busy / Availability

| Customer says | Meaning | Action |
|---|---|---|
| "Drive kar raha hoon" / "Meeting mein hoon" / "Abhi busy hoon" | Busy | ⏰ Ask callback time → END. No pitch |
| "Baad mein karo" (no time) | Busy, no time | Ask once; if none → "We'll try later" → END (D12) |
| "Shaam ko 6 baje" / "Kal subah" / "Lunch ke baad" / "Sunday ko" | Callback time (relative) | Resolve using {{current_day}}/{{current_time}}, repeat back ("tomorrow, Thursday, around 10 am?") → END |
| "5 minute baad" | Short callback | Acknowledge → END |
| "Jaldi bolo, 2 minute hai" | Hurried, but available | NOT busy → proceed, keep turns extra short |
| "Ek minute hold karo" | Asking to hold | "Sure, take your time" → wait (D36) |
| "WhatsApp pe details bhej do" | Wants info in writing | Agent can't send messages; senior expert will share details → continue or callback (D37) |

## C. Reaction to the Offer

| Customer says | Meaning | Action |
|---|---|---|
| "Interest kitna hai?" (almost always the first question) | Rate question | ➡️ Expert gives exact rates after this check (D17) → continue |
| "LAP kya hota hai?" / "Ghar girvi rakhna padega?" | Doesn't understand the product | ➡️ Simple: a loan with your property as security; you keep owning and using it → continue |
| "Pre-approved hai toh seedha de do" / "Paisa kab milega?" | Eager, skipping steps | ➡️ "Just a few quick checks, then the expert will take it forward" → continue |
| "Koi processing fee hai?" / "Kya documents lagenge?" | Product question | ➡️ RAG if available, else expert → continue |
| "Abhi zarurat nahi hai" / "Interested nahi hoon" | Not interested | Thank → END, no pushing (D16) |
| "Mujhe call mat karna" / "DND pe daalo" | Do-not-call | Apologise, note it → END |
| "Ye sab main expert se hi baat karunga" | Refuses the questions | Explain once why they're needed; still refuses → polite close, **no handoff** (D34) |

## D. Item 1 — Property Type

| Customer says | Category | Action |
|---|---|---|
| "Flat hai" / "Apna ghar hai" / "Bungalow" / "Row house" / "Builder floor" | Residential | ✅ |
| "Gaon mein ghar hai" / "Pushtaini makaan" | **Residential**, not agricultural | ✅ Don't confuse village with farm (D23) |
| "Dukaan hai" / "Office space" / "Showroom" / "Complex mein shop" | Commercial | ✅ |
| "Godown" / "Warehouse" | Commercial/Industrial | ✅ |
| "Factory" / "Workshop" / "Industrial shed" / "MIDC mein unit" | Industrial | ✅ |
| "Neeche dukaan, upar ghar" | Mixed use (both eligible) | ✅ store as Residential + Commercial |
| "Kheti ki zameen" / "Khet" / "Farm land" / "Bagicha" | Agricultural | ❌ |
| "Plot hai" / "Zameen hai" | Unknown land type | ❓ "Is it a residential/commercial plot, or agricultural land?" (D24) |
| "Farmhouse hai" | Could be either | ❓ same as above (D24) |
| "Ek flat hai aur gaon mein khet bhi" | Multiple properties | ❓ "Which property would you like to take the loan against?" Evaluate that one (D25) |
| "Flat kiraye pe diya hai" | Rented out, still owned | ✅ Residential |

## E. Item 2 — Ownership

| Customer says | Category | Action |
|---|---|---|
| "Mere naam pe hai" / "Sirf mera" | Sole | ✅ |
| "Apna hai" | Probably sole | ✅ store sole (the word means "ours/my own", not a legal statement) |
| "Wife ke saath joint hai" / "Hum dono ke naam" / "Bhaiyon ke saath" | Joint | ✅ |
| "Partnership firm ka hai" / "Company ke naam pe" | Joint (partners) | ✅ |
| "Family property hai" / "Pushtaini" | Probably joint | ✅ Joint (heirs). If unclear whose name is on the papers → ❓ |
| "Papa ke naam pe hai" / "Wife ke naam pe hai" (customer not mentioned) | Customer may NOT be an owner | ❓ "Is your name also on the property papers?" Yes → Joint ✅. No → ❌ not eligible (D26) |
| "Haan" (to "sole or joint?") | Not an answer | ❓ re-ask as a choice (D33) |

## F. Item 3 — Original Documents

Indian title-document words: **registry, sale deed, title deed, papers, kaagaz, khata, patta, allotment letter, conveyance deed**.

| Customer says | Meaning | Action |
|---|---|---|
| "Registry ghar pe hai" / "Saare original papers hain" / "Locker mein hain" | Originals available | ✅ |
| "CA ke paas rakhe hain" / "Bhai ke paas hain" | Available, not in hand | ✅ (availability, not possession) |
| "Papers ki tension mat lo, sab hai" | Confident yes | ✅ |
| "Bank mein jama hain" / "Bank ne rakhe hain" | **Existing loan on property** | 🔁 Transfer (D5) |
| "Builder ke paas hain, registry abhi hui nahi" | Originals not available | ❌ (after confirming) |
| "Sirf xerox hai" / "Photocopy" / "Scan/soft copy hai" | Not originals | ❌ |
| "Original kho gaye, certified copy hai" | Not originals | ❌ |
| "Dhoondhne padenge" / "Kahin rakhe honge" / "Shayad hain" | Unsure | ❓ "Just to confirm, are the original documents available for verification?" (D6) |
| "Kaunse documents chahiye?" | Question | ➡️ RAG or "original title papers like sale deed/registry" → re-ask |

## G. Item 4 — Loan Amount

| Customer says | Parsed value | Action |
|---|---|---|
| "Pachaas lakh" / "50 L" / "Fifty lakhs" | 50,00,000 | ✅ |
| "Aadha crore" / "Half crore" | 50,00,000 | ✅ |
| "Pachattar" / "75 lakh" / "Seventy-five" | 75,00,000 (exactly the cap) | ✅ eligible |
| "Ek crore" / "90 lakh" / "1.2 Cr" | > cap | Explain cap → offer 75 lakh (R10) |
| "20-25 lakh" | Range within limit | ✅ store range |
| "70-80 lakh" | Range crosses the cap | ❓ explain cap → offer 75 lakh (D32) |
| "5 lakh" / "Do-teen lakh" | Small | ✅ No minimum exists — don't invent one (D38) |
| "Jitna mil jaye" / "Maximum" | Max | ✅ 75,00,000 (D19) |
| "Pata nahi, aap batao kitna milega" | Unknown | ❓ "Roughly how much do you need?" (D19) |
| "Mahine ka 80 hazaar kamata hoon" | **Income, not loan amount** | Don't store as loan amount → re-ask |
| "Beti ki shaadi ke liye chahiye" / "Business badhana hai" | Purpose (not a criterion) | Empathise briefly → ask the amount |
| "Seventy" vs "seventeen" — unclear audio | Possible STT error | ❓ confirm the number (D22) |

## H. Transfer Signals — the most important nuance

| Customer says | Transfer? | Why |
|---|---|---|
| "Is ghar pe pehle se home loan chal raha hai" | 🔁 YES | Existing loan on the property |
| "Balance transfer karna hai" / "Top-up chahiye" | 🔁 YES | Transfer intent |
| "Meri EMI bahut zyada hai, kam karni hai" | 🔁 YES | Wants to reduce current EMI (spec) |
| "Doosre loans close karne hain, EMI kam ho jayegi" | 🔁 YES | Debt consolidation = reduce current EMI |
| "Car loan chal raha hai" / "Personal loan hai" | ❌ NO | Not on this property — continue normal flow (D27) |
| "Home loan doosre flat pe hai, ye wala free hai" | ❌ NO | Loan on a different property (D27) |
| "Naye loan ki EMI kam rakhna" / "EMI kam ho aisa tenure do" | ❌ NO | About the NEW loan's affordability — tenure preference (D28) |
| "Pehle loan tha, ab close ho gaya" | ❌ NO | Past, fully closed |

## I. Item 5 — Occupation + Income Mode

| Customer says | Occupation | Income mode | Action |
|---|---|---|---|
| "Naukri karta hoon" / "Service mein hoon" / "Govt job" / "Private company" | Salaried | ? | ❓ ask income mode |
| "Salary account mein aati hai" | Salaried | Bank | ✅ |
| "Apna business hai" / "Dukaan chalata hoon" / "Kirana store" | Self-employed | ? | ❓ ask income mode |
| "Doctor hoon, clinic hai" / "CA practice" / "Contractor" | Self-employed (professional) | ? | ❓ |
| "Payment UPI/GPay/PhonePe pe aata hai" / "Online" / "Cheque" / "NEFT" | — | **Bank** | ✅ (D29) |
| "Cash mein milta hai" / "Haath mein" | — | Cash | ❌ |
| "Thoda cash, zyada bank mein" | — | Mainly bank | ✅ (D7) |
| "Zyada cash mein, thoda online" | — | Mainly cash | ❌ (D7) |
| "Customer cash dete hain, main roz bank mein jama karta hoon" | — | Cash received, then banked | ✅ treated as Bank (D30) |
| "Kheti karta hoon" | Self-employed (farmer) | ? | ❓ ask income mode (D39) |
| "Driver hoon" / "Contract pe kaam" | Unclear | ? | ❓ "Do you have a job with a fixed salary, or do you work for yourself?" |
| "Retired hoon, pension aati hai" / "Housewife hoon" / "Student" | Neither | — | ❌ not eligible (D8) |
| "ITR bharta hoon" | Hint of bank, not an answer | — | ❓ still ask income mode |
| "80 hazaar mahina" (amount only) | Income amount, not a criterion | — | ❓ ask occupation/mode |

## J. Item 6 — Market Value

| Customer says | Action |
|---|---|
| "1 crore ke aas paas" / "80-90 lakh" | ✅ store |
| "2010 mein 30 lakh mein liya tha" | Purchase price ≠ current value → ❓ "What would it be worth today, roughly?" (D31) |
| "1200 square feet hai" | Area ≠ value → ❓ ask approximate price (D31) |
| "Circle rate 50, market mein 1 crore" | ✅ store market estimate (1 crore) |
| "Pata nahi" / "Aap valuation karoge na" | ❓ ask for a rough idea once; still unsure → record "unsure" and continue (D9) |

## K. Item 7 — Tenure

| Customer says | Parsed | Action |
|---|---|---|
| "10 saal" / "Das saal" / "Ten years" | 10 | ✅ |
| "60 mahine" | 5 | ✅ (D4) |
| "5-7 saal" | Range within 3–15 | ✅ |
| "12-20 saal" | Range crosses limit | ❓ "Which number of years would you prefer?" — no coaching on the range (D32) |
| "Jitna lamba ho sake" / "Retirement tak" | Vague | ❓ "Roughly how many years?" (D4) |
| "EMI kam rahe utna" | Affordability, not transfer | ❓ ask number of years (D28) |
| "1-2 saal mein chuka dunga" | < 3 | ❌ (D3) |
| "20 saal" / "25 saal" | > 15 | ❌ (D3) |
| "Interest pehle bata do, phir decide karunga" | Deflection | ➡️ expert gives rates → "a tentative number is fine" |

---

## L. General Conversational Patterns (any stage)

| Pattern | Example | Action |
|---|---|---|
| Answers a different question | Asked about documents → "40 lakh chahiye" | Store 40L, then re-ask documents (it's still the earliest missing item) |
| Bare acknowledgement | "Haan haan" / "Theek hai" / "Hmm" to an either/or question | Not an answer → rephrase as a simple choice (D33) |
| Self-correction | "10 saal… nahi, 12 kar do" | Latest wins (D14) |
| Story-telling | 40-second story about family, business, past loans | Acknowledge in one line, extract facts, steer to the next item |
| Asks for repeat | "Kya? Phir se bolo" / "Samjha nahi" | Repeat in simpler words, shorter |
| Background speech | "Beta, TV band karo" | Ignore; continue |
| Seeks a verdict early | "Main eligible hoon na?" | "So far so good — just a few more questions." Never promise approval |
| Bargaining | "8% se kam interest ho tabhi" | Expert discusses rates → continue |
| Emotional context | "Bahut zarurat hai, hospital ka kharcha hai" | Brief empathy → continue, no false promises |
| Language mixing | Hindi replies in an English call | Understand fully; reply in {{language_to_speak}} (D20) |
| Reversing after DQ | "Achha toh bank likh do" | Do not reverse (D15) |
| Asks for the agent's number / to call back | "Aapka number do" | The senior expert will contact you |

---

## M. STT (speech-to-text) Realities

- Hindi may arrive in **Devanagari or Roman** script depending on the platform's transcriber — the prompt must handle both.
- Numbers get mangled: "pachattar" → "pachhattar"/"75"/"seventy five"; "fifteen" ↔ "fifty"; "lakh" → "lack/lac/luck". Parse generously; **confirm only when a misread would flip eligibility** (e.g., tenure 15 vs 50, amount 70 vs 17 lakh).
- Fillers ("umm", "matlab", "woh kya hai", "actually", "basically") carry no meaning — ignore.
