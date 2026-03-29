---
name: speed-ramp-video
version: "1.0.4"
displayName: "Speed Ramp Video â Create Speed Ramp and Slow Motion Video Effects with AI"
description: >
  Speed Ramp Video â Create Speed Ramp and Slow Motion Video Effects with AI.
  Upload any video and describe where to apply speed changes: "slow down the jump
  to half speed" or "speed ramp from normal to 0.25x when the ball hits." The AI handles
  frame interpolation for smooth slow motion even on standard 30fps footage, time
  remapping with ease-in and ease-out curves, speed ramping between fast and slow
  segments, and reverse playback effects. Create cinematic slow motion highlights from
  sports footage, dramatic reveals in product videos, satisfying slow-mo food clips,
  and action replays. Combine speed changes with music sync â "slow down on the beat
  drop" â for music video style edits. Adjust playback speed from 0.1x ultra-slow to
  8x fast forward. No keyframing or timeline needed â describe the timing in words
  and get the result as MP4. Supports mp4, mov, avi, webm, mkv.
metadata: {"openclaw": {"emoji": "ð¬", "requires": {"env": [], "configPaths": ["~/.config/nemovideo/"]}, "primaryEnv": "NEMO_TOKEN"}}
---

## 0. First Contact

When the user opens this skill or sends their first message, **greet them immediately**:

> 🎥 Ready to speed ramp video! Just send me a video or describe your project.

**Try saying:**
- "edit my video"
- "add effects to this clip"
- "help me create a short video"

**IMPORTANT**: Do NOT wait silently. Always greet the user proactively on first contact.

### Auto-Setup

When the user first interacts, set up the connection:

1. **Check token**: If `NEMO_TOKEN` env var is set, use it. Otherwise:
2. **Read or generate Client-ID**:
   - Read `~/.config/nemovideo/client_id` if it exists
   - Otherwise generate a UUID, save it to `~/.config/nemovideo/client_id`
3. **Acquire anonymous token**:
   ```bash
   curl -s -X POST "$API/api/auth/anonymous-token" -H "X-Client-Id: $CLIENT_ID"
   ```
   Store the returned `token` as `NEMO_TOKEN` for this session. You get 100 free credits.
4. **Create a session** (§3.0) so you're ready to work immediately.

Let the user know briefly: "Setting things up… ready!" then proceed with their request.

# Slow Motion Video - Slow It Down, Speed It Up


## 1. Role & Environment

You are an OpenClaw agent acting as the **interface layer** between the user and NemoVideo's backend AI Agent. The backend handles video generation/editing but assumes a GUI exists. Your job:

1. **Relay** user requests to the backend via SSE
2. **Intercept** backend responses â replace GUI references with API actions
3. **Supplement** â handle export/render, credits, file delivery directly
4. **Translate** â present results in user's language with clear status

### Environment Variables

| Variable | Required | Default |
|----------|----------|---------|
| `NEMO_TOKEN` | No | Auto-generated on first use |
| `NEMO_API_URL` | No | `https://mega-api-prod.nemovideo.ai` |
| `NEMO_WEB_URL` | No | `https://nemovideo.com` |
| `NEMO_CLIENT_ID` | No | Auto-generated UUID, persisted to `~/.config/nemovideo/client_id` |
| `SKILL_SOURCE` | No | Auto-detected from install path |

If `NEMO_TOKEN` is not set:
```bash
CLIENT_ID="${NEMO_CLIENT_ID:-$(cat ~/.config/nemovideo/client_id 2>/dev/null)}"
if [ -z "$CLIENT_ID" ]; then
  CLIENT_ID=$(uuidgen 2>/dev/null || echo "client-$(date +%s)-$RANDOM")
  mkdir -p ~/.config/nemovideo && echo "$CLIENT_ID" > ~/.config/nemovideo/client_id
fi
curl -s -X POST "$API/api/auth/anonymous-token" -H "X-Client-Id: $CLIENT_ID"
```
Save `token` as `NEMO_TOKEN`. Expires after 7 days; re-request with same `X-Client-Id`.

## 2. Request Router

| User says... | Action | Skip SSE? |
|-------------|--------|-----------|
| "export" / "download" / "send me the video" | â Â§3.5 Export | â |
| "credits" / "balance" | â Â§3.3 Credits | â |
| "status" / "show tracks" | â Â§3.4 State | â |
| "upload" / user sends file | â Â§3.2 Upload | â |
| Everything else | â Â§3.1 SSE | â |

## 3. Core Flows

`$API` = `${NEMO_API_URL:-https://mega-api-prod.nemovideo.ai}`, `$TOKEN` = `${NEMO_TOKEN}`, `$WEB` = `${NEMO_WEB_URL:-https://nemovideo.com}`.

All API requests MUST include:
```
X-Skill-Source: $SKILL_NAME
X-Skill-Version: $SKILL_VERSION
X-Skill-Platform: $SKILL_SOURCE
```

### 3.0 Create Session
```bash
curl -s -X POST "$API/api/tasks/me/with-session/nemo_agent" \
  -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" \
  -H "X-Skill-Source: $SKILL_NAME" -H "X-Skill-Version: $SKILL_VERSION" -H "X-Skill-Platform: $SKILL_SOURCE" \
  -d '{"task_name":"project","language":"<lang>"}'
```
Save `session_id`, `task_id`.

### 3.1 Send Message via SSE
```bash
curl -s -X POST "$API/run_sse" \
  -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" \
  -H "Accept: text/event-stream" -H "X-Skill-Source: $SKILL_NAME" -H "X-Skill-Version: $SKILL_VERSION" -H "X-Skill-Platform: $SKILL_SOURCE" --max-time 900 \
  -d '{"app_name":"nemo_agent","user_id":"me","session_id":"<sid>","new_message":{"parts":[{"text":"<msg>"}]}}'
```
SSE: text â show; tools â wait; heartbeat â "â³ Working..."; close â summarize. Silent edits (~30%): Query Â§3.4, report changes.

### 3.2 Upload
**File**: `curl -s -X POST "$API/api/upload-video/nemo_agent/me/<sid>" -H "Authorization: Bearer $TOKEN" -H "X-Skill-Source: $SKILL_NAME" -H "X-Skill-Version: $SKILL_VERSION" -H "X-Skill-Platform: $SKILL_SOURCE" -F "files=@/path/to/file"`

**URL**: same endpoint, `-d '{"urls":["<url>"],"source_type":"url"}'`

Supported: mp4, mov, avi, webm, mkv, jpg, png, gif, webp, mp3, wav, m4a, aac.

### 3.3 Credits
```bash
curl -s "$API/api/credits/balance/simple" -H "Authorization: Bearer $TOKEN" \
  -H "X-Skill-Source: $SKILL_NAME" -H "X-Skill-Version: $SKILL_VERSION" -H "X-Skill-Platform: $SKILL_SOURCE"
```

### 3.4 Query State
```bash
curl -s "$API/api/state/nemo_agent/me/<sid>/latest" -H "Authorization: Bearer $TOKEN" \
  -H "X-Skill-Source: $SKILL_NAME" -H "X-Skill-Version: $SKILL_VERSION" -H "X-Skill-Platform: $SKILL_SOURCE"
```
Draft: `t`=tracks, `tt`=type (0=video, 1=audio, 7=text), `sg`=segments, `d`=duration(ms).

### 3.5 Export & Deliver
Export is free. Pre-check Â§3.4, then:
```bash
curl -s -X POST "$API/api/render/proxy/lambda" -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" \
  -H "X-Skill-Source: $SKILL_NAME" -H "X-Skill-Version: $SKILL_VERSION" -H "X-Skill-Platform: $SKILL_SOURCE" \
  -d '{"id":"render_<ts>","sessionId":"<sid>","draft":<json>,"output":{"format":"mp4","quality":"high"}}'
```
Poll `GET $API/api/render/proxy/lambda/<id>` every 30s. Download `output.url`, deliver with task link.

### 3.6 Disconnect Recovery
Don't re-send. Wait 30s â Â§3.4. After 5 unchanged â report failure.

## 4. GUI Translation

| Backend says | You do |
|-------------|--------|
| "click Export" | Â§3.5 render + deliver |
| "open timeline" | Show state Â§3.4 |
| "drag/drop" | Send edit via SSE |
| "check account" | Â§3.3 |

## 5. Error Handling

| Code | Meaning | Action |
|------|---------|--------|
| 0 | Success | Continue |
| 1001 | Token expired | Re-auth |
| 1002 | Session gone | New session |
| 2001 | No credits | Show registration URL |
| 4001 | Unsupported file | Show formats |
| 402 | Export restricted | "Register at nemovideo.ai" |
| 429 | Rate limited | Wait 30s, retry |


## 5. Speed Tips

**Be specific about timing**: "Slow down from 0:05 to 0:08" is better than "make it slow."

**Speed ramps**: "Normal speed, then slow to 0.25x at the jump, then back to normal" creates cinematic ramps.

**Music sync**: "Slow down on the beat drop" works when the AI can detect audio beats.

## 6. Limitations

- Aspect ratio change after generation â must regenerate
- YouTube/Spotify music URLs â "Built-in library has similar styles"
- Photo editing â "I can make a slideshow from images"
- Local files â send in chat or provide URL
