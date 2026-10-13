---
name: hindi-video-dub
description: English to Hindi video dubbing. Gemini + edge-tts + ffmpeg.
---

# English → Hindi Video Dub

## Workflow (6 steps)

### 1. Extract audio
```bash
ffmpeg -i video.mp4 -ar 16000 -ac 1 audio.wav  # Gemini needs 16kHz mono
ffmpeg -i video.mp4 -vn -ar 44100 -ac 2 -b:a 128k audio.mp3
```

### 2. Transcribe with Gemini (timestamps!)
Use `~/transcribe_gemin.py` — Gemini 2.5-flash transcribes audio + returns JSON with segments [{start, end, text}]. API key: GEMINI_API_KEY in ~/.ig_env.

### 3. Translate to Hindi (BATCH in ONE API call)
Use `~/translate_hindi.py`: Send ALL segments in one prompt → Gemini returns array.
- **DO NOT** translate per-segment — hits 429 rate limit.
- Voice style: natural documentary narrator Hindi, Devanagari script.

### 4. Generate Hindi TTS per segment
```bash
edge-tts --text "<hindi>" --voice hi-IN-MadhurNeural --write-media seg_XXX.mp3  # Male
edge-tts --text "<hindi>" --voice hi-IN-SwaraNeural --write-media seg_XXX.mp3   # Female
```

### 5. Speed-sync to match video

**KEY INSIGHT:** Natural Hindi is ~1.34x longer than English (37 segs: 177s vs 132s).

**Best approach (user preference):**
- **Video 1.1x slower:** `setpts=1.1*PTS` (132s → 145s)
- **Audio atempo ~1.22x:** 177s → 145s (mild speed-up, natural sounding)

**Alternative (fully natural):**
- Video 0.74x slower (132s → 177s), audio natural — but video very slow

### 6. Merge
```bash
# Slow video first
ffmpeg -y -i video.mp4 -filter:v "setpts=1.1*PTS" -c:v libx264 -preset veryfast -crf 23 -an video_slow.mp4

# Adjust audio
ffmpeg -y -i hindi_combined.mp3 -filter:a "atempo=1.22" hindi_adj.mp3

# Merge
ffmpeg -y -i video_slow.mp4 -i hindi_adj.mp3 -map 0:v -map 1:a -c:v copy -c:a aac -shortest final.mp4
```

## Voice Selection

| Voice | Style |
|---|---|
| `hi-IN-MadhurNeural` | Male, warm documentary |
| `hi-IN-SwaraNeural` | Female, bright conversational |

## User Preferences

- **Video 1.1x slower** (natural Hindi voice, mild audio atempo 1.22x)
- Voice quality > perfect sync
- Male voice (Madhur) primary

## Common Bugs

1. **Gemini transcription gives one giant segment** (e.g. 41s) — re-run or split manually.
2. **Gemini 429 rate limit** — batch translate, don't call per-segment.
3. **`/tmp/` writes fail on Termux** — always use `$HOME/` paths.
4. **ffprobe duration parse** — use `-of default=noprint_wrappers=1:nokey=1` to get bare number.

## Files

Working dir: `~/Hermes_Almari/03_DOWNLOADS/`
Scripts: `~/make_slow_video.py`, `~/gen_hindi_segments.py`, `~/merge_hindi.py`, `~/transcribe_gemin.py`, `~/translate_hindi.py`

## Output Variants

- `england_history_hindi.mp4` — 1.0x video, natural Hindi (fast voice)
- `england_history_hindi_slow.mp4` — 0.74x video, fully natural voice
- `england_history_hindi_1_1x_m.mp4` — **1.1x video, mild audio speedup** (user preferred)
- `england_history_hindi_1_1x_f.mp4` — same with female voice
