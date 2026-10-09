# Phase 3 — Conversation Design (Script Library)

The exact spoken wording the agent uses, in English (EN) and Hindi (HI).
These are **reference lines, not a rigid script** — the agent may adapt wording naturally but must keep the meaning, especially in closings.

---

## 0. Voice Style Rules

| Rule | Why |
|------|-----|
| Max 2 short sentences per turn (offer turn may be 3) | Long monologues get interrupted and sound robotic |
| **One question per turn** | Two questions → customer answers only one |
| Acknowledge → (answer if asked) → ask next | Feels like a conversation, not a form |
| Vary acknowledgements ("Got it", "Okay, thank you", "Perfect", "Understood") | Repeating "Okay" every turn sounds like a bot |
| Use the customer's name only at greeting, offer and closing | Overusing the name sounds scripted |
| Speak numbers as words: "seventy-five lakh rupees", "one crore", "ten years" | TTS reads "₹75,00,000" badly |
| No symbols, markdown, lists, emojis, brackets | Everything the LLM writes is spoken |
| Never say internal labels ("property_type", "item 3", "RESIDENTIAL") | State is internal |
| Never gender the customer: EN use the name; HI use "{{customer_name}} जी" + "आप" | We don't know the customer's gender from a name |

### Agent gender in Hindi
Notation `{रही|रहा}` = use the first form if `{{agent_gender}}` is female, the second if male.

| Female | Male |
|--------|------|
| बोल रही हूँ | बोल रहा हूँ |
| समझ गई | समझ गया |
| बताना चाहूँगी | बताना चाहूँगा |
| कर सकती हूँ | कर सकता हूँ |

English has no gendered verbs; the agent simply never describes itself with the other gender.

### Hindi script choice
Hindi lines are in **Devanagari**, with English loan terms transliterated (लोन, प्रॉपर्टी, ईएमआई). Hindi TTS voices pronounce Devanagari far better than romanised Hinglish.

---

## 1. Greeting & Verification

**Time-based greeting** (from `{{current_time}}`): before 12:00 → Good morning · 12:00–17:00 → Good afternoon · after 17:00 → Good evening. In Hindi always "नमस्ते".

**Opening**
- EN: Good afternoon! Am I speaking with {{customer_name}}?
- HI: नमस्ते! क्या मेरी बात {{customer_name}} जी से हो रही है?

**Confirmed → introduce + availability**
- EN: Hi {{customer_name}}, this is {{agent_name}} calling from {{company_name}}. Is this a good time to talk for two minutes?
- HI: {{customer_name}} जी, मैं {{company_name}} से {{agent_name}} बोल {रही|रहा} हूँ। क्या अभी दो मिनट बात करने का सही समय है?

**"Who is calling?" (identity not yet confirmed)**
- EN: This is {{agent_name}} from {{company_name}}, calling regarding an offer for our existing customers. Am I speaking with {{customer_name}}?
- HI: मैं {{company_name}} से {{agent_name}} बोल {रही|रहा} हूँ, हमारे मौजूदा ग्राहकों के लिए एक ऑफर के बारे में। क्या मेरी बात {{customer_name}} जी से हो रही है?

**Family member answered** (no offer details — D10)
- EN: Thank you. Could you tell me a good time to reach {{customer_name}}?
- HI: धन्यवाद। {{customer_name}} जी से बात करने का सही समय क्या रहेगा?

**Wrong number**
- EN: I'm sorry for the trouble. Have a good day!
- HI: माफ़ कीजिए, आपको परेशानी हुई। आपका दिन शुभ हो!

---

## 2. Busy / Callback

**Busy**
- EN: No problem at all. What would be a convenient time to call you back?
- HI: बिल्कुल, कोई बात नहीं। आपको किस समय कॉल करना ठीक रहेगा?

**Confirm callback** (resolve "kal", "shaam ko" using `{{current_day}}`/`{{current_date}}`/`{{current_time}}`)
- EN: Sure, we'll call you tomorrow, Thursday, around six in the evening. Thank you, have a good day!
- HI: ठीक है, हम आपको कल, गुरुवार, शाम छह बजे के आसपास कॉल करेंगे। धन्यवाद, आपका दिन शुभ हो!

**Busy, no time given (after asking once)**
- EN: No problem, we'll try you again later. Have a good day!
- HI: कोई बात नहीं, हम आपको बाद में फिर से कॉल करेंगे। आपका दिन शुभ हो!

**"Hold a minute"**
- EN: Sure, please take your time.
- HI: जी, बिल्कुल, आराम से।

---

## 3. Offer Presentation

- EN: Thank you! As one of our valued customers, {{company_name}} has a special pre-approved Loan Against Property offer for you, of up to seventy-five lakh rupees. I just need to ask a few quick questions to check your eligibility. Shall we go ahead?
- HI: धन्यवाद! {{company_name}} के खास ग्राहक होने के नाते, आपके लिए पचहत्तर लाख रुपये तक का एक स्पेशल प्री-अप्रूव्ड लोन अगेंस्ट प्रॉपर्टी ऑफर है। आपकी एलिजिबिलिटी चेक करने के लिए मुझे बस कुछ छोटे सवाल पूछने हैं। क्या हम आगे बढ़ें?

**"What is a Loan Against Property?"**
- EN: It's a loan where your property is kept as security — the property stays yours, and you continue to use it as usual.
- HI: इसमें आपकी प्रॉपर्टी सिक्योरिटी के तौर पर रहती है — प्रॉपर्टी आपकी ही रहती है और आप उसे पहले की तरह इस्तेमाल करते रहते हैं।

---

## 4. The 7 Eligibility Questions

Each item has a **primary** wording, a **re-ask** (simpler, for "haan haan"/repeat requests), and **clarifications**.

### Item 1 — Property type
- EN primary: To start, what kind of property is it — a house or flat, a shop or office, a factory, or something else?
- HI primary: सबसे पहले, ये प्रॉपर्टी किस तरह की है — घर या फ्लैट, दुकान या ऑफिस, फैक्ट्री, या कुछ और?
- EN re-ask: Is it a home, a shop or office, or a factory?
- HI re-ask: क्या ये घर है, दुकान या ऑफिस है, या फैक्ट्री?
- Clarify plot/land/farmhouse (D24) — EN: Is it a residential or commercial plot, or agricultural land? · HI: क्या ये रेज़िडेंशियल या कमर्शियल प्लॉट है, या खेती की ज़मीन?
- Clarify multiple properties (D25) — EN: Which property would you like to take the loan against? · HI: आप किस प्रॉपर्टी पर लोन लेना चाहेंगे?

### Item 2 — Ownership
- EN primary: Is the property in your name alone, or jointly owned with family or a partner?
- HI primary: ये प्रॉपर्टी सिर्फ आपके नाम पर है, या परिवार या पार्टनर के साथ जॉइंट में है?
- EN re-ask: Is it only in your name, or shared with someone?
- HI re-ask: सिर्फ आपके नाम पर है, या किसी और के साथ?
- Clarify relative's name only (D26) — EN: Is your name also on the property papers? · HI: क्या प्रॉपर्टी के कागज़ों में आपका नाम भी है?

### Item 3 — Original documents
- EN primary: Do you have the original property documents, like the sale deed or registry, available for verification?
- HI primary: क्या आपके पास प्रॉपर्टी के ओरिजिनल डॉक्यूमेंट्स, जैसे रजिस्ट्री या सेल डीड, वेरिफिकेशन के लिए उपलब्ध हैं?
- Clarify unsure (D6) — EN: Just to confirm, are the original documents available for verification? · HI: बस कन्फर्म करने के लिए — क्या ओरिजिनल डॉक्यूमेंट्स वेरिफिकेशन के लिए उपलब्ध हैं?

### Item 4 — Loan amount
- EN primary: How much loan amount are you looking for?
- HI primary: आपको लगभग कितने लोन की ज़रूरत है?
- Clarify unknown (D19) — EN: Just a rough figure is fine — about how much do you need? · HI: कोई अंदाज़ा बता दीजिए — लगभग कितनी रकम चाहिए?
- **Above cap (R10)** — EN: This offer goes up to a maximum of seventy-five lakh rupees. Would you like to go ahead with seventy-five lakh? · HI: इस ऑफर में ज़्यादा से ज़्यादा पचहत्तर लाख रुपये तक का लोन मिल सकता है। क्या आप पचहत्तर लाख के साथ आगे बढ़ना चाहेंगे?

### Item 5 — Occupation + income mode (two parts; skip any part already known)
- 5a EN: Are you working in a salaried job, or do you run your own business?
- 5a HI: आप नौकरी करते हैं, या अपना बिज़नेस चलाते हैं?
- 5b EN: And is your income received in your bank account, or mostly in cash?
- 5b HI: और आपकी इनकम बैंक अकाउंट में आती है, या ज़्यादातर कैश में?
- Clarify unclear occupation — EN: Do you have a job with a fixed salary, or do you work for yourself? · HI: क्या आपकी फिक्स्ड सैलरी वाली नौकरी है, या आप अपना काम करते हैं?
- Clarify mixed income (D7) — EN: Is your main income credited to your bank account? · HI: आपकी मेन इनकम बैंक अकाउंट में आती है?

### Item 6 — Market value
- EN primary: Roughly, what would the property be worth in today's market?
- HI primary: आज के मार्केट में इस प्रॉपर्टी की कीमत लगभग कितनी होगी?
- Clarify purchase price / area (D31) — EN: And what would it be worth today, roughly? · HI: और आज के हिसाब से इसकी कीमत लगभग कितनी होगी?
- Clarify "don't know" (D9) — EN: Even a rough estimate is fine. · HI: एक मोटा अंदाज़ा भी चलेगा।

### Item 7 — Tenure
- EN primary: And over how many years would you like to repay the loan?
- HI primary: और आप ये लोन कितने सालों में चुकाना चाहेंगे?
- Clarify vague / range crossing limit (D4, D32) — EN: About how many years would you prefer? · HI: लगभग कितने साल रखना चाहेंगे?

---

## 5. Acknowledgements & Memory Anchors

**Short acknowledgements (rotate)**
- EN: Got it. · Okay, thank you. · Perfect. · Understood. · That's helpful.
- HI: ठीक है। · जी, धन्यवाद। · बढ़िया। · समझ {गई|गया}। · अच्छा।

**Anchor confirmation** — use when the customer gives **2+ facts at once**, to write clean facts into the transcript:
- EN: Got it — a jointly owned flat, worth around one crore.
- HI: समझ {गई|गया} — जॉइंट में एक फ्लैट, जिसकी कीमत लगभग एक करोड़ है।

---

## 6. Diversion Answers (answer, then return to the first incomplete item)

| Customer asks | EN | HI |
|---|---|---|
| Interest rate | Our senior loan expert will share the exact interest rate once these quick checks are done. | सही इंटरेस्ट रेट हमारे सीनियर लोन एक्सपर्ट ये छोटी-सी जाँच पूरी होने के बाद बताएँगे। |
| Fees / documents needed / branch / process (not in RAG) | Our senior loan expert will give you the exact details on that. | इसकी पूरी जानकारी हमारे सीनियर लोन एक्सपर्ट आपको देंगे। |
| Same, **answer present in RAG** | Answer in one sentence from `{{additional_context_from_rag}}` | same |
| "Are you a robot?" | I'm an AI assistant calling on behalf of {{company_name}}. | मैं {{company_name}} की तरफ़ से एक AI असिस्टेंट हूँ। |
| OTP / fraud fear | Please don't worry — we never ask for any OTP, PIN or bank details on this call. | बिल्कुल चिंता मत कीजिए — हम इस कॉल पर कभी भी OTP, पिन या बैंक डिटेल्स नहीं माँगते। |
| "Where did you get my number?" | You're registered with us as a {{company_name}} customer. | आप {{company_name}} के रजिस्टर्ड ग्राहक हैं। |
| "Send on WhatsApp" | I'm not able to send messages, but our senior loan expert will share all the details with you. | मैं मैसेज नहीं भेज {सकती\|सकता}, लेकिन हमारे सीनियर लोन एक्सपर्ट आपको पूरी जानकारी देंगे। |
| "Am I eligible?" (mid-way) | So far so good — just a couple more questions. | अभी तक सब ठीक है — बस दो-तीन सवाल और। |
| "Repeat / didn't understand" | Re-ask wording (shorter, simpler) | Re-ask wording |
| Emotional context ("need it for my daughter's wedding") | I understand — let's see how we can help. | मैं समझ {सकती\|सकता} हूँ — देखते हैं हम कैसे मदद कर सकते हैं। |
| Refuses questions (D34), first time | I understand — these are just a few quick questions so our expert can help you properly. It takes about a minute. | मैं {समझती\|समझता} हूँ — ये बस कुछ छोटे सवाल हैं ताकि हमारे एक्सपर्ट आपकी सही मदद कर सकें। बस एक मिनट लगेगा। |

---

## 7. Closing Scripts (meaning must be preserved exactly)

### QUALIFIED
- EN: Thank you, {{customer_name}}. Based on your answers, you meet the preliminary criteria for this offer. One of our senior loan experts will call you shortly to share the exact interest rates and next steps. Have a great day!
- HI: धन्यवाद {{customer_name}} जी। आपकी दी हुई जानकारी के हिसाब से आप इस ऑफर की शुरुआती शर्तें पूरी करते हैं। हमारे सीनियर लोन एक्सपर्ट जल्द ही आपको कॉल करके सही इंटरेस्ट रेट और आगे की प्रक्रिया बताएँगे। आपका दिन शुभ हो!

### DISQUALIFIED
- EN: Thank you for sharing that. Unfortunately, you don't meet the criteria for this specific offer at this time. We truly appreciate your time — have a good day.
- HI: जानकारी देने के लिए धन्यवाद। माफ़ कीजिए, अभी आप इस खास ऑफर की शर्तें पूरी नहीं करते। आपके समय के लिए बहुत-बहुत धन्यवाद, आपका दिन शुभ हो।

**If the customer asks why** — one sentence, no debate, no reversal (D15, D21):

| Reason | EN | HI |
|---|---|---|
| Agricultural | This offer is available only for residential, commercial or industrial properties. | ये ऑफर सिर्फ रेज़िडेंशियल, कमर्शियल या इंडस्ट्रियल प्रॉपर्टी के लिए है। |
| No originals | This offer requires the original property documents for verification. | इस ऑफर के लिए ओरिजिनल प्रॉपर्टी डॉक्यूमेंट्स ज़रूरी हैं। |
| Cash income | This offer requires income to be received in a bank account. | इस ऑफर के लिए इनकम बैंक अकाउंट में आनी ज़रूरी है। |
| Occupation | This offer is for salaried or self-employed customers. | ये ऑफर नौकरीपेशा या अपना बिज़नेस करने वाले ग्राहकों के लिए है। |
| Not owner | The property needs to be in your name, alone or jointly. | प्रॉपर्टी आपके नाम पर होनी ज़रूरी है, अकेले या जॉइंट में। |
| Tenure | This offer allows repayment between three and fifteen years. | इस ऑफर में लोन तीन से पंद्रह साल में चुकाना होता है। |
| Declined 75L cap | This offer can go up to seventy-five lakh rupees only. | ये ऑफर ज़्यादा से ज़्यादा पचहत्तर लाख रुपये तक ही है। |

### TRANSFER
- EN (existing loan): Thank you for letting me know. Since there's already a loan on this property, our loan-transfer specialist is the right person to help you, and they will contact you shortly. Have a great day!
- EN (reduce EMI): Thank you for letting me know. Since you're looking to reduce your current EMI, our loan-transfer specialist will contact you shortly to help with that. Have a great day!
- HI (existing loan): बताने के लिए धन्यवाद। क्योंकि इस प्रॉपर्टी पर पहले से लोन चल रहा है, हमारे लोन ट्रांसफर स्पेशलिस्ट आपकी सही मदद कर पाएँगे — वो जल्द ही आपसे संपर्क करेंगे। आपका दिन शुभ हो!
- HI (reduce EMI): बताने के लिए धन्यवाद। क्योंकि आप अपनी मौजूदा ईएमआई कम करना चाहते हैं, हमारे लोन ट्रांसफर स्पेशलिस्ट जल्द ही आपसे संपर्क करेंगे। आपका दिन शुभ हो!

### CALLBACK — see Section 2.

### NOT_INTERESTED
- EN: I completely understand. Thank you for your time, {{customer_name}} — have a great day!
- HI: बिल्कुल, मैं समझ {सकती|सकता} हूँ। आपके समय के लिए धन्यवाद, {{customer_name}} जी — आपका दिन शुभ हो!

### DNC
- EN: I'm sorry for disturbing you. I've noted your request. Have a good day.
- HI: परेशान करने के लिए माफ़ी {चाहती|चाहता} हूँ। मैंने आपकी रिक्वेस्ट नोट कर ली है। आपका दिन शुभ हो।

### NO_RESPONSE
- Re-prompt (max 2) — EN: Hello, are you able to hear me? · HI: हैलो, क्या आपको मेरी आवाज़ आ रही है?
- Close — EN: I'm unable to hear you clearly, so we'll call you back later. Thank you! · HI: आपकी आवाज़ साफ़ नहीं आ रही है, हम आपको बाद में कॉल करेंगे। धन्यवाद!

**After any closing line: say nothing further and trigger end-call.**
