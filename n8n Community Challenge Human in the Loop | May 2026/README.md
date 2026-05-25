# Relay Content Pipeline
### n8n Community Challenge — Human in the Loop | May 2026
### 100% Free: Groq + Pollinations.ai + Telegram

---

A social media content pipeline that runs like a real agency — three AI agents, three humans reviewing in Telegram, and every rejection actually changes the next output.

Built for the n8n May 2026 HITL Challenge. Zero paid APIs.

---

## The Idea

Most AI content tools are a black box. You prompt it, it gives you something, you either use it or you don't.

This pipeline is different. It mirrors how a real content agency works — a strategist, a creative, and a QA reviewer — except the "people" are AI agents and the humans review their work in Telegram before anything moves forward.

The part that matters: when a reviewer pushes back, their exact words go back into the AI prompt. Sofia doesn't just retry. She reads "too product-focused, I want thought leadership about meeting culture" and generates completely different angles. That's the difference between a feedback loop and a retry button.

---

## How It Flows

```
Brief Intake Form (client, platform, content type, topic)
    ↓
Config — Brand + Brief (single source of truth: voice, specs, audience)
    ↓
Sofia — AI Agent (proposes 3 content angles)
    ↓
Code Node (sanitizes output for Telegram)
    ↓
Telegram HITL #1 — Sofia Review (Approve / Revise with notes)
    ↓ approved
Marcus — AI Agent (writes post copy + image direction)
    ↓
Parse & Generate Image URL (builds Pollinations.ai prompt)
    ↓
Download Generated Image (pulls the actual image)
    ↓
Telegram HITL #2 — Marcus Review (Approve / Revise with notes)
    ↓ approved
Taylor — AI Agent (quality check: 7 criteria, flags issues)
    ↓
Telegram HITL #3 — Taylor Review (Approve / Back to Marcus)
    ↓ approved
Final Output — Post + Image + Quality Review
    ↓
Telegram — Final notification (post ready to publish)
```

**Feedback loops:**
- Sofia loop: Decline → Sofia Pass Feedback → Sofia AI Agent
- Marcus loop: Decline → Marcus Pass Feedback → Marcus AI Agent
- Taylor loop: Decline → Taylor → Marcus Feedback → Marcus AI Agent

---

## The Three Agents

**Sofia — Strategy**
Reads the brief and the full brand guide. Proposes 3 angles, each with a title, hook, rationale, and content direction. On revision, she reads the reviewer's notes via `lastFeedback` and generates angles that directly address the feedback — not variations of the same idea.

**Marcus — Creative**
Takes Sofia's approved angle and writes platform-specific post copy. Respects character limits, hashtag rules, and brand voice per platform. Also writes a detailed image direction that feeds directly into Pollinations.ai. Both copy and image regenerate together when revised.

**Taylor — QA**
Doesn't create anything. Analyses the post against 7 quality criteria: character count, brand voice, platform fit, target audience match, banned language, visual direction quality, and content type alignment. Summarises findings and gives a recommendation before the human makes the final call.

---

## Tech Stack

| Tool | Role | Cost |
|------|------|------|
| n8n (self-hosted) | Workflow engine | Free |
| Groq — Llama 3.3 70B | All 3 AI agents | Free tier |
| Pollinations.ai | Image generation | Free, no signup |
| Telegram Bot | HITL reviews + notifications | Free |
| ngrok | Webhook tunnel | Free tier |

---

## Nodes (33 total)

| Node | Type | Purpose |
|------|------|---------|
| Brief Intake Form | Form Trigger | Entry point — captures client brief |
| Config — Brand + Brief | Set | Single source of truth for all AI agents |
| Sofia — AI Agent | AI Agent | Generates 3 content angles |
| Code in JavaScript | Code | Sanitizes AI output for Telegram |
| Send a text message | Telegram sendAndWait | HITL pause #1 — Sofia reviews |
| Sofia — Decision Router | Switch | Routes Approve / Revise |
| Sofia — Pass Feedback | Set | Injects reviewer notes into next AI run |
| Marcus — AI Agent | AI Agent | Writes post copy + image direction |
| Parse & Generate Image URL | Code | Builds Pollinations.ai URL from image direction |
| Download Generated Image | HTTP Request | Fetches the generated image |
| Marcus: Review | Telegram sendAndWait | HITL pause #2 — Marcus reviews copy + image |
| Marcus — Decision Router | Switch | Routes Approve / Revise |
| Marcus — Pass Feedback | Set | Injects reviewer notes into next AI run |
| Taylor — AI Agent | AI Agent | Quality review against 7 criteria |
| Taylor review | Telegram sendAndWait | HITL pause #3 — Taylor makes final call |
| Taylor — Decision Router | Switch | Routes Approve / Back to Marcus |
| Taylor → Marcus Feedback | Set | Carries Taylor's QA notes to Marcus |
| Final Output | Set | Compiles post + image + quality review |
| Final post | Telegram | Sends approved post notification |
| Groq Chat Model (x3) | Groq LLM | Powers Sofia, Marcus, Taylor |

---

## Setup

### 1. Prerequisites

- n8n running self-hosted (Docker recommended)
- Groq account — free at [console.groq.com](https://console.groq.com)
- Telegram Bot — create via [@BotFather](https://t.me/BotFather), get your chat ID via [@userinfobot](https://t.me/userinfobot)
- ngrok — [ngrok.com](https://ngrok.com)

### 2. Start ngrok + n8n

```bash
# Terminal 1 — start ngrok
ngrok http 5678

# Terminal 2 — start n8n with your ngrok URL
docker run -d --name n8n \
  -p 5678:5678 \
  -e N8N_WEBHOOK_URL=https://your-url.ngrok-free.app \
  -e WEBHOOK_URL=https://your-url.ngrok-free.app \
  -v n8n_data:/home/node/.n8n \
  n8nio/n8n
```

### 3. Import the Workflow

1. Open n8n → Workflows → **+** → Import from File
2. Select `Relay_Content_Pipeline_HITL_Challenge.json`
3. Add credentials:
   - **Groq API** — used for all 3 AI agents (same credential)
   - **Telegram** — bot token for all Telegram nodes (same credential)
4. Update Chat ID in all Telegram nodes with your actual chat ID
5. Activate the workflow

### 4. Configure Your Brand

Open **Config — Brand + Brief** and update:
- `brand_voice` — how your client sounds
- `key_messages` — core messages they always push
- `language_avoid` — banned words list
- `target_audience` — who they're writing for
- Platform tones and specs for LinkedIn, X, Instagram

Every AI agent pulls from this node. To switch clients, only edit this node.

---

## Test Brief

| Field | Value |
|-------|-------|
| Client | Loopin |
| Platform | LinkedIn |
| Content Type | Product announcement |
| Topic Hint | New transcription feature launch |
| Tone Note | Confident and direct, founder's voice |

---

## Important Notes

**ngrok URL changes on restart** — every time you restart ngrok you get a new URL and need to restart the Docker container with the new URL. For a permanent free tunnel, consider Cloudflare Tunnel.

**Pinned data** — during development, pin node outputs to avoid burning API calls on every test. Unpin everything before your final run.

**Image quality** — Pollinations.ai works best with specific, visual prompts. The more concrete Marcus's image direction, the better the output. Tell the AI to describe scenes, not concepts.

---

## Submission

Built for the **n8n Community Challenge — Human in the Loop, May 2026**.

The core design principle: every human decision is meaningful. Reviewers don't just approve or reject — they write why, and the AI reads it. The `lastFeedback` field is what makes this a real feedback loop rather than a retry mechanism.

---

## Author

**Mohamed Hisham** — AI & Automation Engineer, Cairo
Apex Cycle — AI Automation Agency

