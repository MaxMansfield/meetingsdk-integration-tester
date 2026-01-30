# Meeting SDK Integration Tester

A Meeting SDK-based bot for automated integration testing of Zoom SDKs.

This project is a fork of [zoom/meetingsdk-headless-linux-sample](https://github.com/zoom/meetingsdk-headless-linux-sample), modified to serve as a test automation tool for validating other Zoom SDKs (RTMS SDK, Video SDK, etc.).

## Purpose

This bot can:
- Join Zoom meetings programmatically
- Inject known audio/video into meetings
- Be controlled via HTTP API for test automation
- Enable end-to-end testing of SDK data pipelines

## Prerequisites

1. [Docker](https://www.docker.com/) 20.0+
2. [Zoom Account](https://support.zoom.us/hc/en-us/articles/207278726-Plan-Types-)
3. [Zoom Meeting SDK Credentials](#get-your-zoom-meeting-sdk-credentials)
   - Client ID
   - Client Secret

## Quick Start

### 1. Clone the Repository

```bash
git clone git@github.com:MaxMansfield/meetingsdk-integration-tester.git
cd meetingsdk-integration-tester
```

### 2. Download the Zoom Linux SDK

Download the latest version of the Zoom SDK for Linux from the [Zoom Marketplace](https://developers.zoom.us/docs/meeting-sdk/linux/) and place it in the `lib/zoomsdk/` folder.

### 3. Configure the App

```bash
cp sample.config.toml config.toml
```

Edit `config.toml` with your Meeting SDK credentials and meeting information.

### 4. Build and Run

```bash
docker compose build
docker compose up
```

## HTTP API (Planned)

The bot will expose an HTTP API for remote control:

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/join` | POST | Join a meeting |
| `/leave` | POST | Leave current meeting |
| `/status` | GET | Get bot status |
| `/audio/play` | POST | Play audio file into meeting |

## Integration with RTMS SDK

This bot is designed to work with the [Zoom RTMS SDK](https://github.com/zoom/rtms) for integration testing:

```
+------------------+          +------------------+
|  This Bot        |   Zoom   |  RTMS SDK Test   |
|  - Join meeting  |<-------->|  - Receive RTMS  |
|  - Send audio    |  Cloud   |  - Validate data |
+------------------+          +------------------+
```

## Development

See [CLAUDE.md](CLAUDE.md) for detailed development instructions and roadmap.

## Get Your Zoom Meeting SDK Credentials

1. Navigate to [Zoom Developer Portal](https://developers.zoom.us/)
2. Click "Build App" and choose "Meeting SDK"
3. Fill out the app information
4. Copy the Client ID and Client Secret to your `config.toml`

For more information: [Meeting SDK Developer Guide](https://developers.zoom.us/docs/meeting-sdk/developer-accounts/)

## Security

> :warning: **Never commit config.toml to version control** - it contains sensitive credentials

## License

See [LICENSE.md](LICENSE.md)

## Need Help?

- [Developer Support](https://devsupport.zoom.us)
- [Developer Forum](https://devforum.zoom.us)
- [Meeting SDK Documentation](https://developers.zoom.us/docs/meeting-sdk/linux/)
