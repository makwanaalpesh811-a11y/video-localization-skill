---
name: video-localization
description: "Video Localization (DUB) — English → Hindi on Termux"
version: 1.0.0
author: Alpesh Makwana
license: MIT
platforms: [linux]
---

name: video-localization
description: Localize/dub a video to Hindi on Termux.
version: 1.0.0
author: Alpesh Makwana
license: MIT
platforms: [linux, macos]
---

# Video Localization (DUB) — English → Hindi on Termux

Turn a video's spoken content into a Hindi voiceover that stays in sync with the source video. Works fully on Termux with ffmpeg + edge-tts + Gemini. No GPU, no Colab.

## Golden Rule — Never Speed Up the Dub

Hindi runs ~1.2–1.5× longer than English for the same content. The tempting fix — cranking each Hindi segment's TTS up with ffmpeg `atempo` to fit the English segment's duration — produces robotic-sounding Hindi. If you apply `atempo` values above 1.4 the voice sounds chipmunked and users will complain.

The correct fix: **slow down the video to match natural Hindi, not speed up the Hindi to match the video.**

- Dub shorter than video by ≤1.1×: pad with silence, no atempo needed
- Dub longer than video by ≤1.15×: mild audio atempo (0.85–1.15) is tolerable
- Dub longer than video by >1.15×: slow the video with `setpts={factor}*PTS`, keep audio at natural pace

`setpts` on video is lossless on frame timing. `atempo` on audio above 1.5 audibly distorts vowels and consonant clusters — especially in Hindi where aspirated stops and retroflex consonants are speech-critical.

## Stack

| Piece | Tool | Notes |
|---|---|---|
| Video/audio | ffmpeg | `-ar 16000 -ac 1` for mono WAV, ideal for transcription |
| Transcription | Gemini 2.5 Flash (audio input) | 20 MB audio cap, ~120s timeout per call |
| Translation | Gemini 2.5 Flash (text) | 1 call for whole batch, not per-segment — rate limit |
| TTS | edge-tts `hi-IN-MadhurNeural` (male) or `hi-IN-SwaraNeural` (female) | Voice name from `edge-tts --list-voices \| grep hi-IN` |
| Sync | ffmpeg `atempo` (audio) or `setpts` (video) | See math below |
| Merge | ffmpeg `-map 0:v -map 1:a -c:v copy` | Copy video stream, don't re-encode |

API keys are in `~/.ig_env` as `GEMINI_API_KEY=<key>` — parse with `line.startswith('GEMINI_API_KEY')` and `line.split('=')[1].strip()`, never `read().strip().split('=')` (that picks up the next line's key when multiple are present).

## Workflow

1. **Extract audio** at 16 kHz mono wav for transcription fidelity:
   `ffmpeg -i video.mp4 -ar 16000 -ac 1 audio.wav`
   Keep an MP3 version too for size when sending to API.
2. **Transcribe with timestamps** — one Gemini call, ask for JSON with `transcript` + `segments[{start,end,text}]`. Each segment = one sentence with wall-clock seconds. Parse with regex `\{[\s\S]*\}` then `json.loads`.
3. **Translate to Hindi in ONE call** — batch all segment texts into a single prompt returning a JSON array. Per-segment calls hit Gemini's per-minute rate limit (~429) after 10–15 requests.
4. **Generate TTS per segment** — write each to its own mp3. Never `/tmp/*.mp3` on Termux (permission denied); use `$HOME` or `$HERMES_ALMARI` paths.
5. **Concatenate all segments** in order with `ffmpeg -f concat -safe 0 -i list.txt`.
6. **Measure natural total duration** vs original video duration. Compute ratio = natural_dub / video_dur.
7. **Choose sync strategy by ratio** (see table below).
8. **Merge** slowed video + natural (or mildly atempo'd) audio: `ffmpeg -map 0:v -map 1:a -c:v copy -c:a aac -shortest`.

## Sync Decision Table

| Dub / Video ratio | Strategy | Audio atempo | Video setpts factor |
|---|---|---|---|
| <0.90 | Dub shorter than video | 1.0 (natural) | 1.0 (pad audio with silence via `apad`) |
| 0.90–1.15 | Mild audio stretch | 0.85–1.15 | 1.0 |
| 1.15–1.60 | **Slow video, keep audio natural** | 1.0 | 1.15–1.60 |
| >1.60 | Re-chunk source video into segments matching dub | — | — |

When user says "video slow karo" or equivalent, use the 1.15–1.60 row — this is what preserves natural Hindi voice.

## Voice Selection (Hindi)

Two neural Hindi voices available via edge-tts:
- `hi-IN-MadhurNeural` — male, friendly, documentary tone
- `hi-IN-SwaraNeural` — female, bright, conversational

Generate BOTH versions and let user pick. Cost is one batch of edge-tts calls per segment set (~30 seconds).

## Pitfalls

- **Gemini transcription can hallucinate segment boundaries** — one segment may be marked 40s+ while the actual sentence is 3s. Always sanity-check: any segment where `duration > 15s` is suspicious; print and inspect before running TTS. Fix by re-transcribing just the anomalous segment with explicit sentence-by-sentence instruction.
- **Gemini 429 Too Many Requests** after ~10 rapid calls. Sleep 60s between batches, or batch all translations in ONE call rather than per-segment.
- **Gemini 20MB audio cap** — audio above 20 MB in a single base64 inline_data will fail. For long videos, chunk by ~15 min.
- **ffprobe duration parsing**: `ffprobe -show_entries format=duration -of default=noprint_wrappers=1:nokey=1` returns raw `130.74`. Without `:nokey=1` it returns `duration=130.74\n` which crashes `float()`. Always use `:nokey=1`.
- **Termux /tmp writes fail** with `PermissionError` from edge-tts — write to `$HOME` or `$HERMES_ALMARI`, never `/tmp/*.mp3`.
- **atempo range 0.5–2.0** is hard-clamped in ffmpeg. For ratios outside that range, chain two atempo filters (`atempo=2.0,atempo=1.5` = 3.0x) or fall back to `setpts` on video instead.
- **Hindi TTS rate** — default edge-tts Hindi rate is already slower than English. If you apply a `-10%` rate modifier on top, the natural duration balloons to 1.8–2.0× and forces extreme atempo. Keep rate at 0% for localization work.
- **Never re-encode video when merging** — `-c:v copy` preserves the original codec and quality. Re-encoding via libx264 even at crf 23 loses quality and costs ~30s of encode time per 2 min of video.
- **Lip-sync is not achieved** — this workflow replaces audio, not lips. If user wants lip-synced dub, redirect to Colab SadTalker / LivePortrait, don't try on Termux.

## Related Skills

- `youtube-content-pipeline` — download source video and upload result back
- `youtube-full-access` — final upload step
- `hindivlog-gen` — Hindi vlog generation from scratch (no source video), different use case
