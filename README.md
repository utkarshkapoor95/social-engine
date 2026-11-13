# Social Content Engine

A production-grade, human-in-the-loop AI content pipeline that generates complete Instagram posts from a single topic and publishes them autonomously after email-based approval.

**[Try the Live Demo](https://socialengine.autothynk.com)** · **[See Live Output on Instagram](https://www.instagram.com/autothynk.demo/)**

---

## What It Does

A visitor enters a topic (e.g. "Titanic"), target audience, and tone into a web form. The system then:

1. Generates a tailored caption and hashtags using an LLM (Groq API)
2. Creates a custom AI image matching the topic (Cloudflare Workers AI / Flux)
3. Uploads the image to a CDN (Cloudinary) for reliable delivery
4. Emails a full content preview with **Approve / Revise / Reject** buttons (Brevo)
5. On **Approve**: publishes instantly to Instagram via Buffer API
6. On **Revise**: regenerates fresh caption, hashtags, and image, then re-sends for approval
7. On **Reject**: discards the content and ends the workflow

The entire pipeline runs in under 60 seconds from form submission to live Instagram post. Zero manual content creation.

---

## Architecture

```
                                                          ┌──────────────┐
                                                     ┌───>│ Build Buffer │───> Instagram
                                                     │    │   Mutation   │    (@autothynk.demo)
                                                     │    └──────────────┘
┌──────────┐   ┌──────┐   ┌──────────┐   ┌────────┐ │
│ Web Form │──>│ Groq │──>│Cloudflare│──>│ Brevo  │──┤ Approve
│ (Webhook)│   │  LLM │   │Flux + CDN│   │ Email  │  │
└──────────┘   └──────┘   └──────────┘   └────────┘  ├ Revise ──> Loop back to Groq
                                                      │
                                                      └ Reject ──> End
```

**15 n8n nodes** orchestrate the full flow:

| Stage | Node | What It Does |
|-------|------|-------------|
| Ingestion | Webhook, Set Variables | Receives form data (topic, email, audience, tone) and normalizes inputs |
| Generation | Groq API, Parse Response | Calls an open-weight LLM on Groq's LPU hardware to produce a caption and hashtags as structured JSON |
| Image | Cloudflare Flux, Extract, Cloudinary Upload | Generates a topic-matched AI image via Flux, extracts the base64 output, uploads to Cloudinary CDN |
| Approval | Build Email Body, Brevo Send, Wait for Click | Constructs the HTML approval email with image preview and three action buttons, sends via Brevo, pauses workflow |
| Decision | Switch (Approve/Revise/Reject) | Routes based on which button the user clicked in the email |
| Publish | Build Buffer Mutation, POST to Buffer API | On Approve, constructs a GraphQL mutation with caption + image + Instagram metadata, publishes immediately via Buffer |
| Revision | Set Revision Variables | On Revise, increments the attempt counter and loops back to Groq for a fresh generation |

---

## Tech Stack

| Tool | Role |
|------|------|
| **n8n** (self-hosted) | Workflow orchestration, webhook handling, conditional routing |
| **Groq API** | LLM inference for caption and hashtag generation |
| **Cloudflare Workers AI** | Flux image generation model |
| **Cloudinary** | Image CDN and transformation |
| **Brevo** | Transactional email with approval buttons |
| **Buffer API** | Instagram publishing via GraphQL |
| **DigitalOcean** | VPS hosting the n8n instance (Docker) |
| **GitHub Pages** | Hosts this showcase page |

---

## Live Links

| Resource | URL |
|----------|-----|
| Live Demo Page | [socialengine.autothynk.com](https://socialengine.autothynk.com) |
| Instagram Output | [@autothynk.demo](https://www.instagram.com/autothynk.demo/) |
| Agency Website | [autothynk.com](https://autothynk.com) |
| Daily AI Blog | [blog.autothynk.com](https://blog.autothynk.com) |

---

## How the Approval Email Works

The email the user receives contains:

- The AI-generated caption (full text)
- The AI-generated image (embedded)
- Relevant hashtags
- Three buttons linked to unique webhook URLs with signed tokens:
  - **Approve + Post** (green): triggers immediate Instagram publish
  - **Revise** (amber): loops back for a completely fresh generation
  - **Reject** (red): discards content and ends the workflow

Each button URL contains a cryptographic signature to prevent unauthorized actions.

---

## Key Design Decisions

**Why Groq over OpenAI?** Groq runs open-weight models on custom LPU hardware with sub-second latency and a generous free tier (30 req/min). For caption generation where speed matters more than frontier reasoning, Groq is the right tradeoff.

**Why Cloudflare Flux over DALL-E?** Flux runs on Cloudflare's edge network with a free tier of 100 images/day. No API key billing surprises. Image quality is strong for social media use cases.

**Why Buffer over direct Instagram API?** Instagram's Graph API requires a Facebook Business account, app review, and complex token refresh flows. Buffer abstracts all of that into a single GraphQL mutation with a stable access token. Free tier allows 3 channels and 10 scheduled posts/channel.

**Why Brevo over SendGrid/Gmail?** Brevo's free tier is 300 emails/day with full API access including HTML templates. No credit card required. The API key auth is simpler than OAuth flows.

**Why human-in-the-loop?** AI-generated content needs a quality gate before going to a brand's public feed. The Approve/Revise/Reject pattern gives the operator control without requiring them to open n8n or any dashboard. The entire review happens inside their email inbox.

---

## Project Structure

```
social-engine/
├── index.html          # Showcase page (deployed to GitHub Pages)
├── CNAME               # Custom domain config for socialengine.autothynk.com
└── README.md           # This file
```

The n8n workflow JSON runs on a self-hosted n8n instance and is not included in this repo for security (it contains API credentials). The workflow architecture is fully documented above.

---

## Built By

**Utkarsh Kapoor** — AI-native marketer and workflow architect

- Agency: [autothynk.com](https://autothynk.com)
- GitHub: [utkarshkapoor95](https://github.com/utkarshkapoor95)

Part of the [Autothynk](https://autothynk.com) portfolio of AI marketing systems.
