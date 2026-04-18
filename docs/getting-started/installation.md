# Installation

## Go Module

```bash
go get github.com/plexusone/twilio-go
```

## Twilio Credentials

You'll need:

1. **Account SID** - Found on your Twilio Console dashboard
2. **Auth Token** - Found on your Twilio Console dashboard
3. **Phone Number** - A Twilio phone number for sending SMS/making calls

### Environment Variables

```bash
export TWILIO_ACCOUNT_SID="ACxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
export TWILIO_AUTH_TOKEN="your-auth-token"
export TWILIO_PHONE_NUMBER="+15551234567"
```

## Package Imports

Import the packages you need:

=== "SMS (OmniChat)"

    ```go
    import "github.com/plexusone/twilio-go/omnichat"
    ```

=== "Voice (OmniVoice)"

    ```go
    import (
        "github.com/plexusone/twilio-go/omnivoice/callsystem"
        "github.com/plexusone/twilio-go/omnivoice/transport"
        "github.com/plexusone/twilio-go/omnivoice/tts"
        "github.com/plexusone/twilio-go/omnivoice/stt"
    )
    ```

=== "Direct Client"

    ```go
    import "github.com/plexusone/twilio-go/client"
    ```

## Verification

Verify your installation:

```go
package main

import (
    "context"
    "fmt"
    "os"

    "github.com/plexusone/twilio-go/client"
)

func main() {
    c, err := client.New(&client.Config{
        AccountSID: os.Getenv("TWILIO_ACCOUNT_SID"),
        AuthToken:  os.Getenv("TWILIO_AUTH_TOKEN"),
    })
    if err != nil {
        panic(err)
    }

    numbers, err := c.ListPhoneNumbers(context.Background())
    if err != nil {
        panic(err)
    }

    fmt.Printf("Found %d phone numbers\n", len(numbers))
}
```

## Version Compatibility

| twilio-go | omnichat | omnivoice-core | Go |
|-----------|----------|----------------|-----|
| v0.4.x | v0.5+ | v0.7+ | 1.21+ |
| v0.3.x | - | v0.6+ | 1.21+ |
| v0.2.x | - | v0.5+ | 1.21+ |
