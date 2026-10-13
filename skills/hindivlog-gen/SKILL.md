---
name: hindivlog-gen
description: "Hindi daily vlog pipeline: edge-tts + ffmpeg on Termux."
---

# Hindi Vlog Generator (Termux)

Generate 1-minute daily Hindi voiceover vlogs from activity data. Two-stage pipeline: activity log → Hindi TTS → (optional) ffmpeg screen + voice combine.

## Tested Working Pipeline

```
activity_log.py → activity.json          # collect today's Hermes/trading activity
vlog_gen.py --activity → script.txt       # 140-160 Hindi words, ~60s speech
edge-tts → voice.mp3                      # hi-IN-SwaraNeural (female) default
ffmpeg → vlog.mp4                         # screen + voice + text overlay + music
```

Scripts at `/sdcard/Hermes_Almari/01_STRATEGY/code/ig_uploader/vlogs/`:
- `activity_log.py` — scans `~/Hermes_Almari/` for recent files, reads signal_bot states, bot status
- `vlog_gen.py` — Hindi script generator + TTS + ffmpeg combine

## Generate a Daily Vlog

```bash
BASE=/sdcard/Hermes_Almari/01_STRATEGY/code/ig_uploader

# 1. Log today's activity
python3 $BASE/vlogs/activity_log.py --pretty

# 2. One-shot (screen recording required)
python3 $BASE/vlogs/vlog_gen.py \
  --screen recordings/today.mp4 \
  --activity vlogs/activity_$(date +%F).json \
  --out vlogs/vlog_$(date +%Y%m%d).mp4

# Or audio-only (no screen)
python3 $BASE/vlogs/vlog_gen.py \
  --activity vlogs/activity_$(date +%F).json \
  --out vlogs/vlog_$(date +%Y%m%d).mp4
```

## Required Setup

```bash
pkg install ffmpeg                    # aarch64, already installed on Termux
pip install edge-tts                  # free MS neural TTS, no API key
```

## Voices (Hindi)

| Voice | Gender | Flag |
|-------|--------|------|
| `hi-IN-SwaraNeural` | Female | `--voice hi-IN-SwaraNeural` |
| `hi-IN-MadhurNeural` | Male | `--voice hi-IN-MadhurNeural` |

List all: `edge-tts --list-voices | grep hi-IN`

**Default: `SwaraNeural`** — friendly, positive female tone, best for daily vlogs. Override per-call with `--voice`.

## Script Length Rules

- **1 minute ≈ 140-160 Hindi words** at normal pace
- Hook in first 125 chars (before "...more")
- No AI-slop words — use `ig-human` skill's detect.py to score before recording
- CTA at end: subscribe/like/next video tease

## Publishing

Vlogs upload to Instagram Reels via Meta Graph API v23 — see `ig_uploader/` tool:

```bash
python3 /sdcard/Hermes_Almari/01_STRATEGY/code/ig_uploader/ig_uploader.py full vlog.mp4 --caption "..."
```

Requires `IG_ACCESS_TOKEN` + `IG_USER_ID` in `~/.ig_env` (60-day token, 6 permissions). Meta Dev Portal setup documented in `ig_uploader/README.md`.

## Character-style Voice Tuning

When user wants an anime/character voice (e.g. "Hinata voice"), real voice cloning is not feasible on Termux:
- PyTorch has no wheels for Python 3.14 on this device (checked).
- librosa / scipy fail to build from source on Termux.
- Voice-clone models (RVC, Coqui XTTS, OpenVoice V2, GPT-SoVITS) all require PyTorch and 2-4 GB free RAM; phone has <1.5 GB available.
- Paid alternatives exist (ElevenLabs $5/mo, HuggingFace Spaces free tier) but require network signups the user may not want.

**What works: edge-tts parameter tuning.** Generate a batch of 8-10 combinations in ONE pass, send all samples to the user, let them pick. Do NOT guess a single answer — the user has strong taste and one-shot guessing produces rejections.

Tested Hinata-style sweet spot (soft, gentle, slightly shy):
- Voice: `en-US-AriaNeural` (English female works better for anime tone than `hi-IN-SwaraNeural`; Hindi neural voices are tuned for news/read speech, not soft character dialogue)
- Rate: `-20%` to `-25%` (slow)
- Pitch: `+25Hz` to `+30Hz` (young, higher)

Batch generator pattern:

```bash
BASE=/sdcard/Hermes_Almari/01_STRATEGY/code/ig_uploader
TEXT="Hello everyone! How are you? I have some news to share with you."
combos=(
  "en-US-AriaNeural -10% +15Hz"
  "en-US-AriaNeural -15% +20Hz"
  "en-US-AriaNeural -20% +25Hz"
  "en-US-AriaNeural -20% +30Hz"
  "en-US-AriaNeural -25% +30Hz"
  "en-US-JennyNeural -15% +20Hz"
  "en-US-JennyNeural -20% +25Hz"
  "ja-JP-NanamiNeural -15% +15Hz"
  "ja-JP-NanamiNeural -20% +20Hz"
)
i=1
for combo in "${combos[@]}"; do
  set -- $combo
  edge-tts --text "$TEXT" --voice $1 --rate $2 --pitch $3 \
    --write-media "$BASE/audio/search_$(printf '%02d' $i).mp3"
  i=$((i+1))
done
```

Then send every `search_XX.mp3` as MEDIA: attachments in a single reply with voice/rate/pitch labels. Mark top 3 recommendations inline.

Pitfalls:
- **Never attempt voice-clone install on Termux as a first move** — PyTorch install fails silently, wastes 3+ minutes, and the model won't fit in RAM even if it installed. State the constraint upfront.
- **Never deliver a single sample and ask "is this good?"** — user will say bekar and the session stalls. Always batch-generate 5-10 options.
- **English Aria voice is the closest free match to anime-girl tone**, even for Hindi vlogs — Hindi neural voices sound too formal/news-anchored. If user needs Hindi text with anime tone, use `en-US-AriaNeural` and accept the accent, OR mix Hindi text with English-friendly word choices.
- **Copyright honest note**: if user asks for a specific character voice (©-owned), state the risk in one line (personal non-monetized = low risk, monetized = medium, commercial = high). Don't lecture, don't refuse.

## Pitfalls

- **Termux blocks writing to `/tmp` for TTS** — always write to `~/Hermes_Almari/...` or workspace dir. `edge-tts --write-media /tmp/foo.mp3` fails with `PermissionError`. Mechanism: Termux sandbox denies non-filesystem writes.
- **Activity JSON path doubling** — `activity_log.py` uses `os.path.dirname(__file__)` for output dir, so if the script is inside `vlogs/`, output goes to `vlogs/vlogs/`. Fix by passing explicit `--out` or editing `OUTPUT_DIR`.
- **1-minute pacing is tight** — SwaraNeural speaks ~155 wpm Hindi. Above 160 words, expect overflow past 60 sec. Count words BEFORE recording; trim if over.
- **Devanagari in TTS**: edge-tts handles Hindi script natively (no transliteration needed), but Devanagari inside shell args can break on some locales. Pass script via stdin or `--script "$(cat file.txt)"` if inline fails.
- **ffmpeg fontfile for text overlay** — use `/system/fonts/Roboto-Regular.ttf` on Termux (system fonts dir). Custom fonts require explicit path; `drawtext` fails silently without a valid fontfile.
- **Screen recording gap** — Termux has no native screen capture. Use phone's built-in screen recorder, or `scrcpy --record=out.mp4` (USB + PC required). For fully automated fallback, generate screenshot slideshow instead of screen recording.
