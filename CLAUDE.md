# Meeting SDK Integration Tester

A Meeting SDK-based bot for automated integration testing of Zoom SDKs (RTMS SDK, Video SDK, etc.).

## Purpose

This project provides a test bot that can:
1. Join Zoom meetings programmatically
2. Inject known audio/video into the meeting
3. Be controlled via HTTP API for test automation
4. Enable other SDKs (like RTMS SDK) to connect and validate received data

## Origin

This project is a **fork** of [zoom/meetingsdk-headless-linux-sample](https://github.com/zoom/meetingsdk-headless-linux-sample).

**Why fork (not submodule/clone):**
- Can pull upstream changes when Meeting SDK sample is updated
- Can PR improvements back to the original sample
- Clear lineage/history of modifications

## Architecture

```
+-------------------------------------------------------------+
|  Test Environment                                           |
|                                                             |
|  +------------------+          +------------------+         |
|  |  This Bot        |          |  SDK Under Test  |         |
|  |                  |          |  (e.g., RTMS)    |         |
|  |  - Join meeting  |   Zoom   |  - Receive data  |         |
|  |  - Send audio    |<-------->|  - Validate      |         |
|  |  - HTTP control  |  Cloud   |                  |         |
|  +------------------+          +------------------+         |
|           |                              |                  |
|           +-------------+----------------+                  |
|                         v                                   |
|                 +------------------+                        |
|                 |  Test Harness    |                        |
|                 +------------------+                        |
+-------------------------------------------------------------+
```

## Key Components from Base Sample

| Directory | Purpose | Modifications Needed |
|-----------|---------|---------------------|
| `src/Zoom.cpp` | Meeting SDK wrapper, join/leave | Expose via HTTP API |
| `src/raw_send/` | Video injection | Add audio injection |
| `src/raw_record/` | Audio/video recording | May not need |
| `src/events/` | Meeting callbacks | Connect to HTTP API |
| `Dockerfile` | Container build | Keep as-is |
| `compose.yaml` | Docker orchestration | Modify for test use |

## Current State

The base sample has:
- Meeting join/leave via CLI or config file
- Video source injection (`raw_send/ZoomSDKVideoSource`)
- Audio recording (`raw_record/`)
- Event handling for meeting callbacks
- Docker containerization

**What needs to be added:**
1. HTTP Control API for remote automation
2. Audio injection capability (send audio files into meeting)
3. Test asset management (known audio/video files)

## Modifications Roadmap

### Priority 1: HTTP Control API

Add `src/http_api/` for remote control:

```cpp
// Endpoints to implement:
POST /join           - Join a meeting
POST /leave          - Leave meeting
GET  /status         - Bot status (in meeting, idle, etc.)
POST /audio/play     - Play audio file into meeting
POST /video/play     - Play video file (optional)
```

Implementation approach:
- Use lightweight HTTP library (cpp-httplib or similar)
- Add HTTP server thread in `main.cpp`
- Expose Zoom.cpp methods via HTTP endpoints

### Priority 2: Audio Injection

The base sample has video injection via `raw_send/ZoomSDKVideoSource`. Need to add audio:

```cpp
// New file: src/raw_send/ZoomSDKAudioSource.cpp
// Implement IZoomSDKVirtualAudioMicEvent to send audio data
```

Audio format considerations:
- WAV files for simplicity
- Convert to format Meeting SDK expects
- Handle timing/pacing of audio playback

### Priority 3: Test Assets

Create `test_assets/` directory with:
- `test-speech.wav` - Audio with known phrases for transcript validation
  - Suggested phrases: "Hello world. Testing one two three. Integration test."
- Duration: 10-15 seconds
- Format: PCM 16-bit, 16kHz mono (typical for speech)

### Priority 4: Integration with RTMS SDK

The RTMS SDK repo (https://github.com/zoom/rtms) will:
1. Start this bot via Docker Compose
2. Create a meeting via Zoom API
3. Tell bot to join meeting (via HTTP)
4. Wait for RTMS webhook
5. Connect RTMS SDK
6. Tell bot to play audio
7. Validate RTMS SDK receives data
8. Clean up

## Build & Run

### Prerequisites
- Docker 20.0+
- Zoom Meeting SDK credentials (Client ID, Client Secret)
- Zoom Meeting SDK files in `lib/zoomsdk/`

### Local Development

```bash
# Copy sample config
cp sample.config.toml config.toml

# Edit config with your credentials
# Add: client_id, client_secret, meeting info

# Build and run
docker compose build
docker compose up
```

### Download SDK Files

The Zoom Meeting SDK files are NOT included in this repo. Download from:
https://developers.zoom.us/docs/meeting-sdk/linux/

Place in `lib/zoomsdk/`:
- `libmeetingsdk.so`
- Header files

## Configuration

### Environment Variables
```bash
ZM_SDK_CLIENT_ID=     # Meeting SDK Client ID
ZM_SDK_CLIENT_SECRET= # Meeting SDK Client Secret
HTTP_PORT=8081        # Control API port (to be added)
LOG_LEVEL=info        # Logging verbosity
```

### Config File (config.toml)
```toml
[sdk]
client_id = "your_client_id"
client_secret = "your_client_secret"

[meeting]
# Option 1: Join URL
join_url = "https://zoom.us/j/123456789?pwd=xxx"

# Option 2: Meeting ID + Password
meeting_id = "123456789"
password = "abc123"
```

## HTTP API Reference (Planned)

### Join Meeting
```bash
curl -X POST http://localhost:8081/join \
  -H "Content-Type: application/json" \
  -d '{"meeting_id": "123456789", "password": "abc123"}'
```

### Leave Meeting
```bash
curl -X POST http://localhost:8081/leave
```

### Get Status
```bash
curl http://localhost:8081/status
# Response: {"status": "in_meeting", "meeting_id": "123456789"}
```

### Play Audio
```bash
curl -X POST http://localhost:8081/audio/play \
  -H "Content-Type: application/json" \
  -d '{"file": "/app/test_assets/test-speech.wav"}'
```

## Project Structure

```
meetingsdk-integration-tester/
├── CLAUDE.md              # This file - context for development
├── README.md              # User-facing documentation
├── Dockerfile
├── compose.yaml
├── sample.config.toml
├── CMakeLists.txt
├── vcpkg.json             # C++ dependencies
├── bin/
│   └── entry.sh           # Container entry point
├── lib/
│   └── zoomsdk/           # SDK files (not in repo)
├── src/
│   ├── main.cpp           # Entry point - add HTTP server here
│   ├── Zoom.cpp           # Meeting SDK wrapper
│   ├── Zoom.h
│   ├── Config.cpp         # Configuration loading
│   ├── Config.h
│   ├── events/            # Meeting event handlers
│   ├── raw_send/          # Video/audio injection
│   │   ├── ZoomSDKVideoSource.cpp
│   │   └── ZoomSDKVideoSource.h
│   ├── raw_record/        # Recording capabilities
│   └── util/              # Utility functions
└── test_assets/           # NEW: Test audio/video files
    └── test-speech.wav
```

## Development Tasks

### Immediate
- [ ] Understand existing codebase (Zoom.cpp, events/, raw_send/)
- [ ] Identify best approach for HTTP server (cpp-httplib, etc.)
- [ ] Research audio injection with Meeting SDK

### HTTP Control API
- [ ] Add HTTP server library to vcpkg.json
- [ ] Create `src/http_api/HttpServer.cpp`
- [ ] Implement /join endpoint
- [ ] Implement /leave endpoint
- [ ] Implement /status endpoint
- [ ] Add HTTP server startup to main.cpp

### Audio Injection
- [ ] Create `src/raw_send/ZoomSDKAudioSource.cpp`
- [ ] Implement IZoomSDKVirtualAudioMicEvent
- [ ] Add /audio/play endpoint
- [ ] Create test audio file
- [ ] Test audio plays in meeting

### Test Integration
- [ ] Create compose.test.yaml for integration
- [ ] Document ngrok setup for webhooks
- [ ] Create example test script

## Dependencies to Add

Consider adding to `vcpkg.json`:
- `cpp-httplib` - Lightweight HTTP server
- Or `drogon` - Full-featured web framework
- Audio processing library if needed

## Notes

### AssemblyAI/Anthropic Integrations
The original sample has AssemblyAI and Anthropic API integrations for transcription and AI. These are **NOT needed** for integration testing and can be removed or ignored.

### Security
- Never commit `config.toml` with real credentials
- Use environment variables in CI/CD
- The `.gitignore` already excludes `config.toml`

### Testing Considerations
- Each test run should use a fresh meeting
- Clean up meetings after tests
- Consider rate limits on Zoom API
- Webhook delivery may have delays

## Related Projects

- **RTMS SDK**: https://github.com/zoom/rtms - Main consumer of this test bot
- **Original Sample**: https://github.com/zoom/meetingsdk-headless-linux-sample
- **Meeting SDK Docs**: https://developers.zoom.us/docs/meeting-sdk/linux/

## Getting Help

- Meeting SDK issues: https://devforum.zoom.us
- This project issues: Check repo issues or contact maintainer
