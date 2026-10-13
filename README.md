# 🎤 Video Localization Skill

English to Hindi video dubbing, anime vlogs, Hindi content pipeline

**Repository by:** [Alpesh Makwana](https://github.com/makwanaalpesh811-a11y)

---

## 📦 What's Inside

4 Hermes Agent skills for video localization skill:

- `anime-vlog-workflow` — Generate Hinata anime vlogs with Jenny voice.
---

# Anime Vlog Workflow — Hinata Character

## C...
- `hindi-video-dub` — English to Hindi video dubbing. Gemini + edge-tts + ffmpeg.
---

# English → Hindi Video Dub

## ...
- `hindivlog-gen` — "Hindi daily vlog pipeline: edge-tts + ffmpeg on Termux."
---

# Hindi Vlog Generator (Termux)

G...
- `video-localization` — Localize/dub a video to Hindi on Termux.
---

# Video Localization (DUB) — English → Hindi on Ter...

---


## 🔐 SECURITY NOTICE

**This repo is safe to use publicly.** It contains only skill definitions (SKILL.md) and documentation - no live credentials.

### What NOT to commit:
- API keys (GitHub, YouTube, Gemini, OpenAI, etc.)
- AWS credentials (`~/.aws/credentials`)
- Telegram bot tokens
- Exchange API keys (Delta, Hyperliquid, etc.)
- Private keys or certificates

### Where to store secrets (on your machine):
```
~/.aws/credentials          # AWS
~/.youtube_token.json       # YouTube OAuth
~/.ig_env                   # Instagram/Gemini
~/.signal_bot_env           # Trading bot
~/.gh_token                 # GitHub token
```

All skills are designed to read secrets from your filesystem, not embed them.


## 🚀 Installation

### Option 1: Clone to Hermes skills directory
```bash
git clone https://github.com/makwanaalpesh811-a11y/video-localization-skill.git ~/.hermes/skills/video-localization-skill
```

### Option 2: Install individual skills
```bash
# Create skills dir if not exists
mkdir -p ~/.hermes/skills

# Clone only what you need
git clone --depth 1 --filter=blob:none https://github.com/makwanaalpesh811-a11y/video-localization-skill.git /tmp/video-localization-skill
cp -r /tmp/video-localization-skill/skills/* ~/.hermes/skills/
```

---

## ⚙️ Prerequisites

Check each skill's `SKILL.md` for specific dependencies. Common ones:
- **Termux** (Android terminal) or Linux/macOS
- **Python 3.11+**
- **Node.js + npm** (for CLI-based skills)
- **API keys** stored in `~/.<name>_env` or `~/.<name>_token.json` files

---

## 🔧 Setup Guide

1. **Install Hermes Agent:** `pip install hermes-agent` or see [Hermes docs](https://hermes-agent.nousresearch.com/docs)
2. **Clone this repo** into `~/.hermes/skills/`
3. **Configure credentials** in your environment (see each skill's SKILL.md)
4. **Use in chat:** Ask Hermes to use any skill by name

---

## 📚 Skill Documentation

Each skill lives in `skills/<skill-name>/SKILL.md` with:
- **YAML frontmatter:** name, description
- **Commands:** CLI/API references
- **Examples:** Ready-to-use snippets
- **Gotchas:** Real-world pitfalls

---

## 🛡️ Security

## 🔐 SECURITY NOTICE

**This repo is safe to use publicly.** It contains only skill definitions (SKILL.md) and documentation - no live credentials.

### What NOT to commit:
- API keys (GitHub, YouTube, Gemini, OpenAI, etc.)
- AWS credentials (`~/.aws/credentials`)
- Telegram bot tokens
- Exchange API keys (Delta, Hyperliquid, etc.)
- Private keys or certificates

### Where to store secrets (on your machine):
```
~/.aws/credentials          # AWS
~/.youtube_token.json       # YouTube OAuth
~/.ig_env                   # Instagram/Gemini
~/.signal_bot_env           # Trading bot
~/.gh_token                 # GitHub token
```

All skills are designed to read secrets from your filesystem, not embed them.

---

## 📜 License

MIT License - see [LICENSE](./LICENSE)

---

## 👨‍💻 Author

**Alpesh Makwana** — Trading, content, and Hermes Agent enthusiast.

- **GitHub:** [makwanaalpesh811-a11y](https://github.com/makwanaalpesh811-a11y)
- **Telegram:** @alpeshalgobot

---

## 🤝 Contributing

Issues and PRs welcome. Follow the SKILL.md format conventions.

## 🏷️ Topics
`hindi` `dubbing` `video` `localization` 
