# 📱 WhatsApp Growth Agent V1

An AI-powered WhatsApp lead qualifier built with **n8n + Groq (LLaMA 70B) + Supabase**. Automatically qualifies incoming WhatsApp leads by asking 4 smart questions conversationally — in Hindi, English, or Hinglish — and scores them as hot/warm/cold.

> Built by [keviv777](https://github.com/keviv777) as part of the [india-ai-agent-toolkit](https://github.com/keviv777/india-ai-agent-toolkit)

---

## 🧠 How It Works

```
[WhatsApp Webhook]
 │
 ▼
[Is Verification?]
 ├── YES ──► [Respond Challenge]   (Meta webhook verify)
 └── NO
      │
      ▼
 [Parse Payload]  →  extract phone, name, message
      │
      ▼
 [Upsert Lead]    →  create or update lead in Supabase
      │
      ▼
 [Load History]   →  fetch last 10 messages from Supabase
      │
      ▼
 [Save User Message]  →  store incoming message
      │
      ▼
 [Claude Qualifier]   →  LLaMA 3.3 70B via Groq API
      │
      ▼
 [Parse Response]     →  extract reply, score, action
      │
      ▼
 [Save Assistant Message]  →  store AI reply in Supabase
      │
      ▼
 [Send WhatsApp Reply]     →  deliver via Meta Graph API
```

---

## ✨ Features

- ✅ **Full webhook verification** — handles Meta's `hub.challenge` flow
- ✅ **Persistent conversation memory** — stores full chat history in Supabase
- ✅ **Multilingual** — responds in Hindi, English, or Hinglish based on user's language
- ✅ **Smart lead scoring** — hot / warm / cold based on budget, timeline, need, decision-making authority
- ✅ **Action routing** — `reply_only`, `book_call`, or `nurture`
- ✅ **Zero OpenAI cost** — uses Groq's free-tier LLaMA 3.3 70B

---

## 🔧 Tech Stack

| Tool | Purpose |
|------|---------|
| n8n (self-hosted) | Workflow orchestration |
| Groq API (LLaMA 3.3 70B) | AI lead qualification |
| Supabase (PostgreSQL) | Conversation history + lead storage |
| WhatsApp Business API | Message send/receive via Meta Graph API |
| JavaScript (Code nodes) | Payload parsing + JSON handling |

---

## 🗄️ Supabase Schema Required

```sql
-- Leads table
CREATE TABLE leads (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  phone TEXT UNIQUE NOT NULL,
  name TEXT,
  score TEXT DEFAULT 'unknown',
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Conversations table
CREATE TABLE conversations (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  lead_phone TEXT NOT NULL,
  role TEXT NOT NULL,  -- 'user' or 'assistant'
  message TEXT NOT NULL,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Upsert function
CREATE OR REPLACE FUNCTION upsert_lead(p_phone TEXT, p_name TEXT)
RETURNS void AS $$
BEGIN
  INSERT INTO leads (phone, name)
  VALUES (p_phone, p_name)
  ON CONFLICT (phone) DO UPDATE SET name = EXCLUDED.name;
END;
$$ LANGUAGE plpgsql;
```

---

## 🚀 Setup Instructions

### 1. Prerequisites
- n8n running (self-hosted or cloud)
- Supabase project created
- Meta WhatsApp Business API access
- Groq API key (free at console.groq.com)

### 2. Import Workflow
1. Download `WhatsApp_Growth_Agent_V1.json`
2. In n8n → **Import from file**
3. Replace all placeholder values (see below)

### 3. Configure Credentials

Replace these placeholders in the workflow nodes:

| Placeholder | Where to get it |
|------------|----------------|
| `YOUR_WHATSAPP_TOKEN_HERE` | Meta Developer Console → WhatsApp → API Setup |
| `YOUR_PHONE_NUMBER_ID` | Meta Developer Console → WhatsApp → Phone Numbers |
| `YOUR_SUPABASE_ANON_KEY` | Supabase → Project Settings → API |
| `https://YOUR_PROJECT.supabase.co` | Supabase → Project Settings → API |
| `YOUR_GROQ_API_KEY` | console.groq.com → API Keys |

### 4. Set Webhook in Meta

1. Go to Meta Developer Console → Webhooks
2. Set webhook URL to: `https://your-n8n-domain/webhook/whatsapp`
3. Subscribe to `messages` field
4. Use the challenge verification — the workflow handles it automatically

---

## 📊 Lead Scoring Logic

The AI asks 4 questions one at a time:
1. **Budget?**
2. **Timeline?**
3. **What do they need?**
4. **Are they the decision maker?**

| Score | Criteria |
|-------|---------|
| 🔥 **Hot** | Budget 20k+, timeline < 1 month, clear need, decision maker |
| 🌡️ **Warm** | Partial match on 2-3 criteria |
| ❄️ **Cold** | Doesn't match most criteria |

---

## 📁 Files

```
whatsapp-growth-agent/
├── WhatsApp_Growth_Agent_V1.json   # n8n workflow (import this)
└── README.md
```

---

## ⚠️ Important Notes

- Never commit real API keys — use environment variables or n8n credentials manager
- Rotate your Supabase anon key if it was ever exposed publicly
- This workflow is for **educational and personal use**

---

## 📄 License

MIT — free to use, modify, and build upon.
