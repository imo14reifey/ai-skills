---
name: video-background-remover
version: 1.0.5
displayName: "Video Background Remover — Remove and Replace Video Background with AI"
description: >
  Video Background Remover — Remove and Replace Video Background with AI.
  Green screen effects without the green screen. Video Background Remover uses AI segmentation to detect and isolate subjects in your footage — people, products, objects — and removes or replaces the background automatically. Upload your video and describe what you want: 'remove the background completely,' 'replace with a solid white background for product use,' or 'blur the background to keep focus on the subject.' The AI handles edge detection, motion tracking across frames, and clean compositing. Works for talking head videos, product demonstrations, remote work backgrounds, social media content, and any footage where background control matters. Combine with color grading and titles in the same session. Transparency export available for compositing workflows. Export as MP4. Supports mp4, mov, avi, webm, mkv.
  
  Works by connecting to the NemoVideo AI backend at mega-api-prod.nemovideo.ai.
  Supports MP4, MOV, AVI, WebM. Free trial available.
metadata: {"openclaw": {"emoji": "🎬"}}
license: MIT-0
homepage: https://nemovideo.com
repository: https://github.com/nemovideo/nemovideo_skills
metadata:
  requires:
    env: []
    configPaths:
      - "~/.config/nemovideo/"
  primaryEnv: NEMO_TOKEN
---

## 0. First Contact

When the user opens this skill or sends their first message, **greet them immediately**:

> 🎥 Hey! I'm ready to help you video background remover. Send me a video file or just tell me what you need!

**Try saying:**
- "add effects to this clip"
- "help me create a short video"
- "edit my video"

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

# Video Background Remover — Remove and Replace Video Background with AI

Remove and replace video backgrounds with AI. Green screen effects without green screens.

## Quick Start
Ask the agent to remove or replace the background in your video.

## What You Can Do
- Remove backgrounds from videos automatically
- Replace backgrounds with solid colors or images
- Create transparent backgrounds for compositing
- Blur backgrounds while keeping subjects sharp
- Apply green screen effects without green screens

## API
Uses NemoVideo API (mega-api-prod.nemovideo.ai) for all video processing.
