# hitl-instagram-bot

A human-in-the-loop Instagram content pipeline: trending-topic research feeds
an LLM idea generator, every stage (idea → image prompts → final images)
waits for your approval over WhatsApp before continuing, and approved posts
publish automatically to Instagram.

Runs on n8n. All external APIs used in the default setup are free-tier.

## Pipeline

```
trend scan → idea generation (Groq)  → WhatsApp approval
           → image prompt generation → WhatsApp approval
           → image generation (Pollinations) → WhatsApp approval
           → Instagram publish
```

## Structure

```
hitl-instagram-bot/
├── workflows/
│   ├── 1-main-pipeline.json              # the pipeline above, import into n8n
│   └── 2-whatsapp-webhook-receiver.json  # catches your WhatsApp replies, resumes the paused pipeline
├── prompts/
│   ├── trend-ideation-system-prompt.txt  # editable, loaded at runtime
│   └── carousel-image-prompts.txt        # editable, loaded at runtime
├── .env.example                          # which free-tier keys you need
├── .gitignore
├── README.md
└── SETUP.md                              # full setup walkthrough, start here
```

## Quickstart

1. `npm install -g n8n && n8n start` (or use n8n.cloud)
2. Import both files from `workflows/` into n8n
3. Follow `SETUP.md` for credentials and the WhatsApp approval wiring
4. Run the main pipeline manually once before turning on the schedule trigger

## Stack

- **Orchestration:** n8n
- **LLM:** Groq (Llama 3.3 70B), free tier
- **Image generation:** Pollinations.ai, free and keyless
- **Approvals:** WhatsApp Business Cloud API
- **Publishing:** Instagram Graph API
