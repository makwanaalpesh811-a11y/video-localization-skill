---
name: anime-vlog-workflow
description: Generate Hinata anime vlogs with Jenny voice.
---

# Anime Vlog Workflow — Hinata Character

## Character Setup
- **Name:** Hinata (from Naruto anime, fan-channel — non-monetized)
- **Personality:** Shy, gentle, soft-spoken, encouraging
- **Voice:** Jenny Anime (LOCKED) — `en-US-JennyNeural`, rate `-20%`, pitch `+25Hz`
- **Language:** English (default), Hindi available

## Voice Profile (JSON)
```json
{
  "name": "jenny_anime_final",
  "voice": "en-US-JennyNeural",
  "rate": "-20%",
  "pitch": "+25Hz",
  "description": "Hinata-style anime voice, soft gentle English"
}
```

## File Locations
- Base: `/sdcard/Hermes_Almari/01_STRATEGY/code/ig_uploader/`
- Scripts: `scripts/`
- Audio: `audio/`
- Images: `images/`
- Videos: `videos/`
- Training: `training/`
- API keys: `~/.ig_env` (Gemini key)

## Script Writing Rules (Gentle Shy Character)

### Sentence Length
- Short sentences: 5-10 words (primary)
- Medium: 10-20 words (with pauses)
- NEVER long/winding sentences

### Pause Placement
- Use `(pause)` marker between major sentences
- Use `...` for hesitation mid-sentence
- Pause before speaking to show shyness
- Example: `(pause)` "Hello everyone!"

### Word Choice
- Contractions always (I'm, don't, can't)
- Softening words: "maybe," "perhaps," "I think"
- Hesitation markers: "um," "uh," "oh..."
- Avoid: assertive words, exclamations, complex clauses

### Format
- 1-minute vlog = 150-200 words = 60-75 seconds
- Timestamps: [0-5 sec], [5-50 sec], [50-60 sec]
- Opening: "Hello everyone! I'm Hinata."
- Closing: "See you in next video! Like and subscribe!"

## Voice Generation Command
```bash
edge-tts --text "$SCRIPT" \
  --voice en-US-JennyNeural \
  --rate -20% \
  --pitch +25Hz \
  --write-media output.mp3
```

## Image Generation (Free: Pollinations.ai)
- URL: `https://image.pollinations.ai/prompt/$ENCODED?width=1024&height=1024&model=flux&nologo=true`
- Rate limited — try single requests, not bulk
- Alternative: Leonardo.ai (150 free/month)

## API Limits (Free Tier)
- **Gemini text:** 5 requests/day (gemini-2.5-flash)
- **Gemini image:** BLOCKED (need paid $19.99/mo)
- **Gemini video:** BLOCKED (need paid)
- **Pollinations.ai:** ~1 req/sec, no hard daily limit
- **edge-tts:** Unlimited

## What Gemini API CAN Do
- ✅ Text generation (scripts, captions, ideas)
- ✅ Knowledge queries (character info, tips)
- ❌ Image generation (quota = 0 on free tier)
- ❌ Video generation (quota = 0)
- ❌ TTS (use edge-tts instead)

## What Cannot Be Done Solo
- ❌ Talking avatar (needs Wav2Lip/SadTalker in Colab)
- ❌ Instagram upload (needs Meta API token from user)
- ❌ Google Colab execution (needs browser login)

## Sample Script Template
```
Hello everyone! I'm Hinata. (pause)
Today was a really productive day... (pause)

So first... (main content 3-4 short paragraphs)

Thanks for watching! If you enjoyed, please like and subscribe. (pause)
See you in the next video!
```

## Complete Pipeline
1. **Script:** Gemini text gen OR manual (English, 150-200 words)
2. **Voice:** edge-tts with Jenny Anime voice
3. **Image:** Pollinations.ai OR existing Hinata image
4. **Avatar:** Wav2Lip Colab (user runs) OR static image
5. **Combine:** ffmpeg (video + voice + music)
6. **Upload:** Meta API (user provides token)

## Scripts to Use
- `gemini_vlog.py` — Full vlog generator
- `activity_log.py` — Day's activity tracker
- `vlog_gen.py` — Voice + video combiner
- `ig_uploader.py` — Meta API Instagram uploader
- `ig_caption_gen.py` — Caption generator

## Copyright Notes
- Hinata = © Naruto (Shueisha/Pierrot)
- Personal fan channel (non-monetized) = LOW risk
- Monetized = MEDIUM risk (DMCA possible)
- Commercial = HIGH risk
- Always credit, never claim as original

## Training Data Location
Full training files saved at:
`/sdcard/Hermes_Almari/01_STRATEGY/code/ig_uploader/training/`

Contains:
- anime_voice_direction.txt — Writing guidelines
- anime_image_prompts.txt — 10 image prompt templates
- anime_vlog_scripts.txt — Script examples
- instagram_anime_captions.txt — Caption templates
- anime_vlog_topics.txt — 30 topic ideas

## Quick Reference
- Voice: Jenny Anime (en-US-JennyNeural -20% +25Hz)
- Length: 60-75 sec (150-200 words)
- Pause marker: `(pause)`
- Hesitation: `...`
- Opening: "Hello everyone!"
- Closing: "See you in next video!"
