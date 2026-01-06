# OBS Studio + Twitch Live Control

This project is struktured's live streaming control center for OBS Studio and Twitch integration, powered by an MCP server that Claude can use to control the stream.

## Quick Start

```bash
# Source credentials first
source setenv.sh

# Run OBS with correct library paths
./run.sh
```

## MCP Server Location

The MCP server is at `mcp-servers/obs-twitch-mcp/`. All stream control goes through this.

## Common Commands (Things struktured often asks)

### BRB (Be Right Back)
When user says "brb" or "back in X":
1. Mute mic: `obs_mute("Mic/Aux", true)`
2. Hide camera: `obs_hide_source("oldcam")`
3. Send chat message: `twitch_send_message("back in 5")`

When user returns ("I'm back"):
1. Unmute mic: `obs_mute("Mic/Aux", false)`
2. Show camera: `obs_show_source("oldcam")`

### Shoutouts
When user says "shoutout to [username]":
- Use `shoutout_streamer(username)` - sends chat message and shows their clip

### Clipping
- **Local clip** (preferred): `obs_clip()` - uses OBS replay buffer, saves locally
- **Twitch clip**: `twitch_create_clip()` - creates clip on Twitch (must be live)

### Raids
- `twitch_raid(username)` - start raid to another channel
- `twitch_find_raid_targets()` - find streamers in same category

### Scene/Source Control
- `obs_switch_scene(scene_name)` - change scene
- `obs_show_source(source_name)` / `obs_hide_source(source_name)`
- `obs_mute(source_name, true/false)`

## Features

### OBS Control
- Scene switching and management
- Source show/hide/edit
- Audio mute/unmute/volume control
- Screenshot capture
- Replay buffer control (start, stop, save clips)
- Recording control (start, stop, pause, resume)
- Browser source management
- Text overlay management

### Twitch Integration
- **Chat**: Send messages, reply to users, read recent messages
- **Moderation**: Ban, timeout, unban users; slow mode; emote-only mode
- **Stream Info**: Get/set title, game/category
- **Raids**: Start raids, find raid targets, cancel raids
- **Clips**: Create clips from live stream, get clip info
- **Shoutouts**: Shoutout streamers with their clips

### Video Uploads
- **YouTube**: Full upload support via Data API v3
  - `upload_video_to_youtube(file_path, title, description, tags, privacy)`
  - `get_my_youtube_videos(count)`
  - Requires OAuth setup (credentials in setenv.sh)
- **Twitch**: No API upload (use Video Producer web UI manually)
- **Generic router**: `upload_video(file_path, platform, title, description)`

### Live Translation
- Capture screenshots for Japanese text OCR
- Display English translation overlay
- `translate_screenshot()` - capture and translate
- `translate_and_overlay(japanese_text, english_text)` - show translation
- `clear_translation_overlay()` - remove overlay

### Alerts & Overlays
- `show_follow_alert(username)` - follower alerts
- `show_custom_alert(title, subtitle, color, duration)` - custom alerts
- `obs_add_text_overlay(text, font_size, color, position)` - text overlays
- `obs_add_browser_source(url, source_name)` - web overlays

### Viewer Interactions
- **Lurk detection**: `show_lurk_animation(username)` when users type !lurk
- **Welcome animations**: Custom HTML animations for viewers (stored in assets/)
- Animations auto-hide after duration or when user returns to chat

## Streaming Setup

### Games Commonly Streamed
- Penta Dragon (1992 Japan-only GB RPG)
- Trinea (action RPG)
- Colorization projects for classic games
- Cowardly Irregular

### Translation Workflow
1. Capture OBS screenshot via `translate_screenshot()`
2. Claude OCRs Japanese text from game
3. Display English translation overlay at bottom of screen
4. Auto-update when dialogue changes

## File Structure

```
mcp-servers/obs-twitch-mcp/
├── src/
│   ├── tools/           # MCP tool definitions
│   │   ├── obs.py       # OBS control tools
│   │   ├── chat.py      # Twitch chat tools
│   │   ├── clips.py     # Clip/replay buffer tools
│   │   ├── uploads.py   # Video upload tools
│   │   ├── translation.py
│   │   ├── alerts.py
│   │   └── shoutout.py
│   └── utils/           # Client implementations
│       ├── obs_client.py
│       ├── twitch_client.py
│       └── youtube_client.py
├── assets/              # HTML overlays and animations
├── setenv.sh            # Credentials (NEVER push to git)
└── .gitignore           # Excludes tokens and user-specific assets
```

## Credentials (setenv.sh)

Required environment variables:
- `TWITCH_CLIENT_ID`, `TWITCH_CLIENT_SECRET`, `TWITCH_OAUTH_TOKEN`
- `TWITCH_CHANNEL` (struktured)
- `OBS_WEBSOCKET_PASSWORD`, `OBS_WEBSOCKET_PORT`
- `YOUTUBE_CLIENT_ID`, `YOUTUBE_CLIENT_SECRET` (for YouTube uploads)

## Important Notes

- Never kill OBS without explicit permission
- Translation overlay: size 80 font, bottom-center, English only
- Twitch channel: struktured
- User-specific assets (welcome animations, profile images) are gitignored
- Token files (.twitch_token.json, .youtube_token.json) are gitignored
- First YouTube upload opens browser for OAuth authorization

## Known Issues / TODO

- **Flaky token auth**: Twitch token authentication can be unreliable. Sometimes requires re-running `uv run python auth.py` or restarting the MCP server. Need to investigate and improve token refresh logic.
- **OBS replay buffer path**: `obs_clip()` sometimes returns empty file_path even when clip saves successfully

## Dependencies

- `obsws-python` - OBS websocket control
- `twitchAPI` - Twitch chat/API integration
- `google-api-python-client` - YouTube uploads
- `pillow` - Image processing
- `httpx` - HTTP client
- Python 3.11+ via uv
