# Setup guide

## 1. Import the workflows
In n8n: **Workflows → Import from File** for both `workflows/1-main-pipeline.json`
and `workflows/2-whatsapp-webhook-receiver.json`. They're separate workflows
on purpose — the receiver has to exist independently so it can wake up the
paused main pipeline whenever a WhatsApp reply arrives.

## 2. Point n8n at this repo's prompts folder
The workflow reads its LLM prompts from `prompts/trend-ideation-system-prompt.txt`
and `prompts/carousel-image-prompts.txt` at runtime, instead of hardcoding
them in the JSON. Set an environment variable n8n can see:

```bash
export PROMPTS_DIR=/absolute/path/to/hitl-instagram-bot/prompts
```

(If running n8n via Docker, pass this as `-e PROMPTS_DIR=... -v /path/to/prompts:/path/to/prompts`.)

This means you can tune your prompts by editing plain text files and
re-running the workflow — no need to touch the JSON or re-import anything.

## 3. Accounts and credentials (all free-tier)

| Service | What for | How to get it |
|---|---|---|
| Groq API key | idea + prompt generation (LLM) | console.groq.com, no card required |
| Pollinations.ai | image generation | nothing to sign up for — keyless |
| WhatsApp Business Cloud API | approval messages | developers.facebook.com → create app → add WhatsApp |
| Instagram Business/Creator account | posting destination | convert your IG account, link to a Facebook Page |
| Instagram Graph API access | posting | same Meta app → add Instagram Graph API, get token with `instagram_content_publish` |
| Airtable (or Postgres/Sheets) | pending-approvals + post log | any lightweight DB works |

Add each as an n8n credential (Settings → Credentials): `groqApi`, `whatsappApi`,
`instagramApi`, `airtableApi` — names must match what's referenced in the JSON.

Copy `.env.example` to `.env` and fill it in as your own reference — n8n
doesn't read this file directly, but it's the single place documenting what
you need, and it's what any scripts in the future would load from.

## 4. The one non-obvious piece: resuming a paused run

n8n's **Wait** node pauses a workflow until a specific webhook URL is called
— that URL is unique per run (`{{$execution.resumeUrl}}`). Your WhatsApp
reply doesn't know that URL on its own, so you bridge them:

1. Right before each Wait node, store a row: `{ from: YOUR_WHATSAPP_NUMBER, resume_url: {{$execution.resumeUrl}}, stage: "idea" }` in Airtable.
2. When your WhatsApp reply arrives, the receiver workflow looks up that row and POSTs your button choice to `resume_url`.
3. The paused Wait node wakes up with your reply, and the following IF node branches to continue or stop.

## 5. Meta / WhatsApp webhook setup

1. Set your Meta app's webhook URL to the receiver workflow's production URL + `/wa-incoming`.
2. Meta sends a GET with `hub.challenge` to verify — handle that in the Webhook node's response.
3. Subscribe to the `messages` field so button replies get delivered.

## 6. Instagram posting prerequisites

- Personal accounts can't post via API — must be Business or Creator.
- Must be linked to a Facebook Page you administer.
- Token needs `instagram_content_publish` scope.
- Carousels: create one container per image (`is_carousel_item: true`), then a parent `CAROUSEL` container, then publish the parent.

## 7. Test order

1. Run the pipeline manually once, confirm the WhatsApp message with buttons arrives.
2. Tap a button, confirm the receiver workflow resumes the paused execution (check n8n's execution log).
3. Only then let the schedule trigger run unattended.
