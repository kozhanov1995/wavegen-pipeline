# 🎵 WAVEGEN — AI Music Video Pipeline

<div align="center">

![WAVEGEN Banner](https://img.shields.io/badge/WAVEGEN-AI%20Pipeline-8b5cf6?style=for-the-badge&logo=youtube&logoColor=white)

**Fully automated AI pipeline that generates and publishes music videos to YouTube Shorts daily — zero manual work.**

[![n8n](https://img.shields.io/badge/n8n-workflow-FF6D5A?style=flat-square&logo=n8n)](https://n8n.io)
[![Claude](https://img.shields.io/badge/Claude-AI-8b5cf6?style=flat-square)](https://anthropic.com)
[![ElevenLabs](https://img.shields.io/badge/ElevenLabs-Audio-000000?style=flat-square)](https://elevenlabs.io)
[![Pexels](https://img.shields.io/badge/Pexels-Video-05A081?style=flat-square)](https://pexels.com)
[![Creatomate](https://img.shields.io/badge/Creatomate-Render-3B82F6?style=flat-square)](https://creatomate.com)
[![YouTube](https://img.shields.io/badge/YouTube-Shorts-FF0000?style=flat-square&logo=youtube)](https://youtube.com/@WAVEGEN)

</div>

---

## ⚡ What It Does

Every day at 10:00 AM the pipeline runs automatically and publishes a new AI-generated music video to YouTube Shorts:

```
Claude AI      →  Generates theme, mood, lyrics, style, subtitles, SEO metadata
ElevenLabs     →  Creates a 30-second music track from the concept
Pexels API     →  Finds matching cinematic vertical background video
Creatomate     →  Renders final 9:16 video: background + audio + animated subtitles
YouTube API    →  Publishes as YouTube Short with SEO title, description, tags
Telegram Bot   →  Sends notification with video link
```

**No server. No FFmpeg. No manual work. Runs entirely in n8n.**

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                         n8n Workflow                            │
│                                                                 │
│  ⏰ Schedule                                                     │
│      │                                                          │
│      ▼                                                          │
│  🧠 Claude API ──► generates concept JSON                       │
│      │                                                          │
│      ▼                                                          │
│  🎵 ElevenLabs ──► generates 30s audio track                    │
│      │                                                          │
│      ├──► ☁️ GoFile ──► temp audio hosting (public URL)         │
│      │                                                          │
│      ▼                                                          │
│  🎬 Pexels API ──► finds portrait cinematic video               │
│      │                                                          │
│      ▼                                                          │
│  🎞️ Creatomate ──► renders: video + audio + subtitles           │
│      │                                                          │
│      ▼                                                          │
│  ⬇️ Download rendered MP4                                       │
│      │                                                          │
│      ▼                                                          │
│  ▶️ YouTube Data API v3 ──► publishes as Short                  │
│      │                                                          │
│      ▼                                                          │
│  📱 Telegram Bot ──► sends notification                         │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🛠️ Tech Stack

| Service | Role | Tier |
|---|---|---|
| **n8n** | Workflow orchestration | Self-hosted / Cloud |
| **Claude API** (claude-sonnet-4-5) | Concept & metadata generation | Pay-per-use ~$0.01/run |
| **ElevenLabs** | AI music/sound generation | Creator plan |
| **Pexels API** | Stock cinematic video | Free |
| **GoFile** | Temporary audio file hosting | Free |
| **Creatomate** | Cloud video rendering | Free tier: 100 renders/mo |
| **YouTube Data API v3** | Auto-publish Shorts | Free |
| **Telegram Bot API** | Run notifications | Free |

**Estimated cost per video: ~$0.02–0.05**

---

## 🧠 Claude Output Schema

Each run Claude generates a structured JSON concept:

```json
{
  "theme": "solitude",
  "mood": "melancholic, introspective",
  "elevenlabs_prompt": "slow piano melody, lo-fi, soft rain ambience, melancholic, 30 seconds",
  "pexels_query": "rainy city night",
  "subtitle_line_1": "some paths are walked",
  "subtitle_line_2": "alone",
  "subtitle_line_3": "and that's okay",
  "youtube_title": "AI Made This Sad Lo-Fi Track 🌧️ #shorts",
  "youtube_description": "Every day I let AI create music.\n\n#aimusic #shorts #lofi #chill",
  "youtube_tags": ["aimusic", "shorts", "lofi", "chill", "viral"]
}
```

---

## 🎬 Video Output Spec

| Parameter | Value |
|---|---|
| Resolution | 1080 × 1920 (9:16) |
| Duration | 30 seconds |
| FPS | 30 |
| Format | MP4 |
| Subtitles | 3 animated lines, fade in/out |
| Audio | AI-generated, fade out at end |
| Video | Looped cinematic stock footage |

---

## 🚀 Setup & Installation

### Prerequisites

- [n8n](https://n8n.io) (self-hosted or cloud)
- API accounts: Anthropic, ElevenLabs, Pexels, Creatomate
- Google Cloud project with YouTube Data API v3
- Telegram Bot (via @BotFather)

### 1. Clone the repo

```bash
git clone https://github.com/kozhanov1995/wavegen-pipeline.git
cd wavegen-pipeline
```

### 2. Configure environment

```bash
cp .env.example .env
```

Fill in your API keys:

```env
ANTHROPIC_API_KEY=sk-ant-...
ELEVENLABS_API_KEY=...
PEXELS_API_KEY=...
CREATOMATE_API_KEY=...
CREATOMATE_TEMPLATE_ID=...
TELEGRAM_BOT_TOKEN=...
TELEGRAM_CHAT_ID=...
```

### 3. Import workflow

1. Open n8n → **Import from file**
2. Select `workflow/wavegen-n8n-workflow-v3.json`
3. Replace all `PASTE_..._HERE` values with your API keys
4. Configure YouTube OAuth2 credential
5. Activate the workflow

### 4. YouTube OAuth2 Setup

1. [Google Cloud Console](https://console.cloud.google.com) → New project
2. APIs & Services → Enable **YouTube Data API v3**
3. Credentials → OAuth 2.0 Client ID → Web Application
4. Add redirect URI: `https://your-n8n-domain/rest/oauth2-credential/callback`
5. In n8n: Credentials → YouTube OAuth2 API → paste Client ID & Secret → Connect

### 5. Creatomate Template

The template is already configured for 9:16 vertical video with:
- Background video layer with loop
- Audio layer with fade out
- 3 animated text subtitle layers

Import `creatomate/template.json` or create manually in Creatomate Dashboard.

---

## 📁 Project Structure

```
wavegen-pipeline/
├── workflow/
│   └── wavegen-n8n-workflow-v3.json   # n8n workflow (import this)
├── creatomate/
│   └── template.json                   # Video template
├── docs/
│   └── screenshots/                    # Pipeline screenshots
├── .env.example                        # Environment variables template
├── .gitignore
└── README.md
```

---

## 🔒 Security Notes

- **Never commit real API keys** — use `.env` files only
- All secrets stored in n8n Credentials (encrypted)
- GoFile links expire after 24h — audio is never permanently stored
- YouTube OAuth2 token stored securely in n8n

---

## 📊 Results

| Metric | Value |
|---|---|
| Time per video | ~4 min (fully automated) |
| Manual work | 0 minutes |
| Cost per video | ~$0.03 |
| Schedule | Daily at 10:00 AM |
| Platform | YouTube Shorts |

---

## 🗺️ Roadmap

- [ ] TikTok cross-posting via Playwright
- [ ] Instagram Reels publishing
- [ ] A/B test Claude prompts for higher engagement
- [ ] Analytics dashboard (views per theme/mood)
- [ ] Multiple videos per day (morning + evening)
- [ ] Custom music styles based on trending topics

---

## 👤 Author

**Dias Kozhanov** — Fullstack Developer

[![GitHub](https://img.shields.io/badge/GitHub-kozhanov1995-181717?style=flat-square&logo=github)](https://github.com/kozhanov1995)
[![Portfolio](https://img.shields.io/badge/Portfolio-kozhanov1995.github.io-8b5cf6?style=flat-square)](https://kozhanov1995.github.io)
[![Email](https://img.shields.io/badge/Email-kozhanov23@yandex.kz-EA4335?style=flat-square&logo=gmail)](mailto:kozhanov23@yandex.kz)

---

<div align="center">

*Built with n8n · Claude API · ElevenLabs · Pexels · Creatomate · YouTube*

**No backend server required.**

</div>
