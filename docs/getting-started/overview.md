# Overview

twilio-go is a Go SDK for Twilio that provides:

1. **Twilio Client** - Low-level REST API client for Twilio services
2. **OmniChat Provider** - SMS messaging via the omnichat `provider.Provider` interface
3. **OmniVoice Providers** - Voice call handling via omnivoice-core interfaces

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│                     Your Application                     │
└─────────────────────────────────────────────────────────┘
                            │
        ┌───────────────────┼───────────────────┐
        │                   │                   │
        ▼                   ▼                   ▼
┌───────────────┐   ┌───────────────┐   ┌───────────────┐
│   omnichat/   │   │  omnivoice/   │   │    client/    │
│  SMS Provider │   │   Providers   │   │  REST Client  │
└───────────────┘   └───────────────┘   └───────────────┘
        │                   │                   │
        └───────────────────┼───────────────────┘
                            │
                            ▼
                   ┌─────────────────┐
                   │   Twilio API    │
                   └─────────────────┘
```

## Use Cases

### SMS Messaging (OmniChat)

Use the `omnichat` package when you need:

- Send/receive SMS messages
- Webhook handling for inbound messages
- Integration with the omnichat multi-provider messaging framework

### Voice Calls (OmniVoice)

Use the `omnivoice` packages when you need:

- Outbound voice calls
- Real-time audio streaming via Media Streams
- Speech-to-text and text-to-speech
- Integration with the omnivoice voice agent framework

### Direct API Access

Use the `client` package when you need:

- Direct access to Twilio REST API
- Custom integrations not covered by the provider interfaces
- Building your own abstractions

## Requirements

- Go 1.21 or later
- Twilio Account SID and Auth Token
- Phone number(s) provisioned in your Twilio account
