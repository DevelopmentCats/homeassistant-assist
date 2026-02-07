# homeassistant-assist

An OpenClaw skill for controlling Home Assistant smart home devices using the Assist (Conversation) API.

## Why?

Instead of the AI manually looking up entity IDs and building service calls, this skill passes natural language directly to Home Assistant's built-in NLU. This is:

- **Faster** — One API call instead of multiple lookups
- **Cheaper** — Fewer tokens spent figuring out entity IDs
- **More reliable** — HA knows your home better than the AI does

## Installation

### From ClawHub
```bash
clawhub install homeassistant-assist
```

### Manual
Clone this repo into your OpenClaw workspace skills directory:
```bash
git clone https://github.com/DevelopmentCats/homeassistant-assist.git ~/.openclaw/workspace/skills/homeassistant-assist
```

## Setup

Add to your OpenClaw config (`~/.openclaw/openclaw.json`):

```json
{
  "env": {
    "HASS_SERVER": "https://your-homeassistant-url",
    "HASS_TOKEN": "your-long-lived-access-token"
  }
}
```

Generate a token: Home Assistant → Profile → Long-Lived Access Tokens → Create Token

## Usage

Just talk naturally:

- "Turn off the kitchen light"
- "Set the thermostat to 72"
- "What lights are on?"
- "Close the garage door"
- "Cycle the litter robot"

The AI will pass your request to Home Assistant's Assist API, which handles entity resolution and executes the command.

## License

MIT
