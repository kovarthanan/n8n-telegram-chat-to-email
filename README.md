# 📨 Telegram → AI → Gmail: Welcome Email Bot (n8n)

An n8n workflow that turns a Telegram message into a personalized welcome email. Send the bot a name and email address, an AI model writes a warm welcome email, and Gmail sends it automatically.

![Workflow canvas](screenshots/workflow-canvas.png)

---

## 🔄 How It Works

```
Telegram Trigger → Basic LLM Chain → Code in JavaScript → Gmail (Send a message)
                         ↑
                 OpenAI Chat Model
```

| Step | Node | What it does |
|------|------|--------------|
| 1 | **Telegram Trigger** | Listens for messages sent to the bot |
| 2 | **Basic LLM Chain** + **OpenAI Chat Model** | Reads the message and writes a welcome email as JSON (`to`, `subject`, `body`) |
| 3 | **Code in JavaScript** | Cleans the AI reply (removes code fences, extracts the JSON) |
| 4 | **Gmail – Send a message** | Sends the email to the extracted address |

### Example

**Telegram message:**
```
Send a welcome email to Priya, priya@example.com
```

**Email received:**
```
Subject: Welcome to Our Community!

Dear Priya,

Welcome! We're thrilled to have you with us. If you need any assistance
or have questions, feel free to reach out.

Best Regards,
Kovarthanan
```

---

## 📁 Project Structure

```
├── workflow/
│   └── telegram-ai-welcome-email.json   # n8n workflow export
├── screenshots/
│   ├── workflow-canvas.png
│   ├── telegram-chat.png
│   └── email-received.png
├── README.md
└── .gitignore
```

---

## 🚀 Setup

### 1. Import the workflow
In n8n, go to **Workflows → ⋯ → Import from File** and select `workflow/telegram-ai-welcome-email.json`.

### 2. Telegram bot
1. In Telegram, message **@BotFather**, send `/newbot` and follow the steps.
2. Copy the bot token.
3. In n8n, open the **Telegram Trigger** node, create a **Telegram API** credential and paste the token.

### 3. OpenAI
1. Create a key at [platform.openai.com/api-keys](https://platform.openai.com/api-keys). The full key is shown **only once**, so copy it immediately.
2. Add credit under **Settings → Billing** (API usage is billed separately from ChatGPT).
3. In n8n, open the **OpenAI Chat Model** node and create a credential with the key.

### 4. Gmail
1. In [Google Cloud Console](https://console.cloud.google.com), create an OAuth client and **enable the Gmail API** (APIs & Services → Library → Gmail API → Enable).
2. Add your Gmail address under **OAuth consent screen → Test users**.
3. In n8n, create a **Gmail OAuth2** credential and sign in with Google.
4. In **Send a message**, set:
   - **To:** `{{ $json.to }}`
   - **Subject:** `{{ $json.subject }}`
   - **Email Type:** Text
   - **Message:** `{{ $json.body }}`

### 5. Run it
Click **Execute workflow** (or activate the workflow), then message your bot.

---

## 🧹 Cleaning the AI Output

LLMs often wrap JSON in ```` ```json ```` fences. The Code node strips them and extracts the object:

```javascript
return $input.all().map(item => {
  let raw = String(item.json.text || item.json.output || '').trim();
  raw = raw.replace(/^```(?:json)?\s*/i, '').replace(/\s*```$/, '');

  const start = raw.indexOf('{');
  const end = raw.lastIndexOf('}');
  if (start !== -1 && end !== -1) raw = raw.slice(start, end + 1);

  try {
    const data = JSON.parse(raw);
    return { json: { to: (data.to || '').trim(), subject: (data.subject || '').trim(), body: (data.body || '').trim() } };
  } catch (e) {
    return { json: { error: 'Could not parse LLM output', raw } };
  }
});
```

---

## 🐞 Troubleshooting

| Error | Fix |
|-------|-----|
| Gmail node fails with empty fields | Map **To**, **Subject** and **Message** to `{{ $json.to }}`, `{{ $json.subject }}`, `{{ $json.body }}` |
| `Forbidden – Gmail API has not been used in project …` | Enable the **Gmail API** in Google Cloud, wait a few minutes, then reconnect the credential |
| `Access blocked: app has not completed verification` | Add your Gmail address as a **Test user** on the OAuth consent screen |
| OpenAI `429 insufficient_quota` | Add credit to your OpenAI API account |
| `to` is empty | The message had no email address. Ask the user for one (e.g. an If node that replies on Telegram) |

---

## 🔒 Security

- API keys and tokens live only in **n8n Credentials**; they are never stored in the workflow JSON.
- The exported JSON contains credential **names and IDs** only, which are not secrets.
- Never commit `.env` files or paste keys into node fields.

---

## 🛠️ Tech Stack

- [n8n](https://n8n.io) – workflow automation
- Telegram Bot API – input channel
- OpenAI (`gpt-4.1-mini`) – email writing
- Gmail API – email delivery
- JavaScript – output cleaning

---

## 🙏 Acknowledgements

Built while learning AI automation with **Social Eagle**.
