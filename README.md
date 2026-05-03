# omni-twilio

[![Go CI][go-ci-svg]][go-ci-url]
[![Go Lint][go-lint-svg]][go-lint-url]
[![Go SAST][go-sast-svg]][go-sast-url]
[![Go Report Card][goreport-svg]][goreport-url]
[![Docs][docs-godoc-svg]][docs-godoc-url]
[![Visualization][viz-svg]][viz-url]
[![License][license-svg]][license-url]

 [go-ci-svg]: https://github.com/plexusone/omni-twilio/actions/workflows/go-ci.yaml/badge.svg?branch=main
 [go-ci-url]: https://github.com/plexusone/omni-twilio/actions/workflows/go-ci.yaml
 [go-lint-svg]: https://github.com/plexusone/omni-twilio/actions/workflows/go-lint.yaml/badge.svg?branch=main
 [go-lint-url]: https://github.com/plexusone/omni-twilio/actions/workflows/go-lint.yaml
 [go-sast-svg]: https://github.com/plexusone/omni-twilio/actions/workflows/go-sast-codeql.yaml/badge.svg?branch=main
 [go-sast-url]: https://github.com/plexusone/omni-twilio/actions/workflows/go-sast-codeql.yaml
 [goreport-svg]: https://goreportcard.com/badge/github.com/plexusone/omni-twilio
 [goreport-url]: https://goreportcard.com/report/github.com/plexusone/omni-twilio
 [docs-godoc-svg]: https://pkg.go.dev/badge/github.com/plexusone/omni-twilio
 [docs-godoc-url]: https://pkg.go.dev/github.com/plexusone/omni-twilio
 [viz-svg]: https://img.shields.io/badge/visualizaton-Go-blue.svg
 [viz-url]: https://mango-dune-07a8b7110.1.azurestaticapps.net/?repo=plexusone%2Fomni-twilio
 [loc-svg]: https://tokei.rs/b1/github/plexusone/omni-twilio
 [repo-url]: https://github.com/plexusone/omni-twilio
 [license-svg]: https://img.shields.io/badge/license-MIT-blue.svg
 [license-url]: https://github.com/plexusone/omni-twilio/blob/master/LICENSE

Go SDK for Twilio with adapters for [OmniChat](https://github.com/plexusone/omnichat) (SMS) and [OmniVoice](https://github.com/plexusone/omnivoice-core) (voice).

## Features

- 📞 **CallSystem**: PSTN call handling (incoming/outgoing phone calls)
- 📡 **Transport**: Twilio Media Streams for real-time audio
- 🗣️ **TTS**: Text-to-speech via Twilio's Say verb (Alice, Polly, Google voices)
- 👂 **STT**: Speech recognition via Gather verb and real-time transcription
- 💬 **SMS**: Send/receive SMS via OmniChat provider interface

## Installation

```bash
go get github.com/plexusone/omni-twilio
```

## Package Structure

```
omni-twilio/
├── client/           # Exported Twilio REST API client
├── omnichat/         # SMS provider for omnichat
└── omnivoice/
    ├── callsystem/   # Call handling provider
    ├── transport/    # Media Streams provider
    ├── stt/          # Speech-to-text provider
    └── tts/          # Text-to-speech provider
```

## Quick Start

### SMS (OmniChat)

```go
import "github.com/plexusone/omni-twilio/omnichat"

provider, _ := omnichat.New(
    omnichat.WithAccountSID("ACxxxxxxxx"),
    omnichat.WithAuthToken("your-token"),
    omnichat.WithPhoneNumber("+15551234567"),
)

// Connect and send SMS
provider.Connect(ctx)
provider.Send(ctx, "+15559876543", provider.OutgoingMessage{
    Content: "Hello from Twilio!",
})

// Handle incoming SMS via webhook
http.Handle("/sms", provider.WebhookHandler())
```

### Voice Calls (OmniVoice)

```go
import (
    "github.com/plexusone/omni-twilio/omnivoice/callsystem"
    "github.com/plexusone/omni-twilio/omnivoice/transport"
)

// Create call system
cs, _ := callsystem.New(
    callsystem.WithPhoneNumber("+15551234567"),
    callsystem.WithWebhookURL("wss://your-server.com/media-stream"),
)

// Handle incoming calls
cs.OnIncomingCall(func(call callsystem.Call) error {
    fmt.Printf("Incoming call from %s\n", call.From())
    return call.Answer(context.Background())
})

// Make outbound call
call, _ := cs.MakeCall(ctx, "+15559876543")
fmt.Printf("Call initiated: %s\n", call.ID())

// Set up webhooks
http.HandleFunc("/incoming", handleIncoming(cs))
http.HandleFunc("/media-stream", handleMediaStream(cs.Transport()))
```

### Direct Client Usage

```go
import "github.com/plexusone/omni-twilio/client"

c, _ := client.New(&client.Config{
    AccountSID: "ACxxxxxxxx",
    AuthToken:  "your-token",
})

// Send SMS
msg, _ := c.SendSMS(ctx, &client.SendSMSParams{
    To:   "+15559876543",
    From: "+15551234567",
    Body: "Hello!",
})

// Make call
call, _ := c.MakeCall(ctx, &client.MakeCallParams{
    To:    "+15559876543",
    From:  "+15551234567",
    Twiml: "<Response><Say>Hello!</Say></Response>",
})
```

### TTS (Text-to-Speech)

```go
import "github.com/plexusone/omni-twilio/omnivoice/tts"

provider, _ := tts.New(
    tts.WithVoice("Polly.Joanna"),
    tts.WithLanguage("en-US"),
)

// Generate TwiML
result, _ := provider.Synthesize(ctx, "Hello!", tts.SynthesisConfig{
    VoiceID: "Polly.Matthew",
})
// result.Audio contains TwiML
```

### STT (Speech-to-Text)

```go
import "github.com/plexusone/omni-twilio/omnivoice/stt"

provider, _ := stt.New(
    stt.WithLanguage("en-US"),
    stt.WithSpeechModel("phone_call"),
)

// Generate TwiML for speech recognition
twiml := provider.GenerateGatherTwiML(stt.GatherConfig{
    Input:         "speech",
    Language:      "en-US",
    SpeechTimeout: "auto",
    Action:        "/handle-speech",
    Prompt:        "Please say your account number",
})
```

### Transport (Media Streams)

```go
import "github.com/plexusone/omni-twilio/omnivoice/transport"

tr, _ := transport.New()

// Listen for Media Stream connections
connCh, _ := tr.Listen(ctx, "/media-stream")

for conn := range connCh {
    go func(c transport.Connection) {
        audio := make([]byte, 1024)
        for {
            n, _ := c.AudioOut().Read(audio)
            // Process audio...
            c.AudioIn().Write(responseAudio)
        }
    }(conn)
}
```

## Configuration

### Environment Variables

```bash
export TWILIO_ACCOUNT_SID="your-account-sid"
export TWILIO_AUTH_TOKEN="your-auth-token"
```

### Available Voices

**Twilio Basic**: `alice`, `man`, `woman`

**Amazon Polly**: `Polly.Joanna`, `Polly.Matthew`, `Polly.Amy`, `Polly.Brian`, etc.

**Google TTS**: `Google.en-US-Standard-A` through `D`, `Google.en-US-Wavenet-A` through `D`

## Testing

```bash
# Unit tests
go test -v ./...

# Integration tests (requires credentials)
export TWILIO_ACCOUNT_SID="ACxxxx"
export TWILIO_AUTH_TOKEN="xxxx"
export TWILIO_PHONE_NUMBER="+15551234567"
go test -v -tags=integration ./...
```

## Migration from omnivoice-twilio

This package was renamed from `omnivoice-twilio` to `twilio-go` in v0.4.0.

| Before | After |
|--------|-------|
| `github.com/plexusone/omnivoice-twilio/callsystem` | `github.com/plexusone/omni-twilio/omnivoice/callsystem` |
| `github.com/plexusone/omnivoice-twilio/transport` | `github.com/plexusone/omni-twilio/omnivoice/transport` |
| `github.com/plexusone/omnivoice-twilio/tts` | `github.com/plexusone/omni-twilio/omnivoice/tts` |
| `github.com/plexusone/omnivoice-twilio/stt` | `github.com/plexusone/omni-twilio/omnivoice/stt` |

New in v0.4.0:

- `github.com/plexusone/omni-twilio/client` - Exported Twilio client
- `github.com/plexusone/omni-twilio/omnichat` - SMS provider for OmniChat

## Related Packages

- [omnivoice-core](https://github.com/plexusone/omnivoice-core) - Voice interfaces
- [omnichat](https://github.com/plexusone/omnichat) - Chat interfaces
- [elevenlabs-go](https://github.com/plexusone/elevenlabs-go) - ElevenLabs SDK

## License

MIT
