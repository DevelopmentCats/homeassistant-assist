---
name: homeassistant-assist
description: Control Home Assistant smart home devices using the Assist (Conversation) API. Use this skill when the user wants to control smart home entities - lights, switches, thermostats, covers, litter robots, vacuums, media players, or any other smart device. Passes natural language directly to Home Assistant's built-in NLU for fast, token-efficient control.
homepage: https://github.com/DevelopmentCats/homeassistant-assist
metadata:
  openclaw:
    emoji: "🏠"
    requires:
      bins: ["curl"]
      env: ["HASS_SERVER", "HASS_TOKEN"]
    primaryEnv: "HASS_TOKEN"
---

# Home Assistant Assist

Control smart home devices by passing natural language to Home Assistant's Assist (Conversation) API. This is the **preferred method** for smart home control — faster and more token-efficient than manually resolving entity IDs.

## When to Use This Skill

Use this skill when the user's intent is to **control or query smart home devices**, including but not limited to:

- **Lights**: "turn off the kitchen light", "dim the bedroom to 50%", "set living room lights to blue"
- **Switches/Plugs**: "turn on the fan", "switch off the office lamp"
- **Climate**: "set thermostat to 72", "turn on the AC"
- **Covers**: "close the garage door", "open the blinds"
- **Media**: "pause the TV", "play music in the living room"
- **Vacuums**: "start the vacuum", "send roomba home"
- **Litter Robots**: "cycle the litter robot", "check litter robot status"
- **Scenes**: "activate movie mode", "turn on the goodnight scene"
- **Queries**: "what lights are on?", "is the front door locked?", "what's the temperature?"

**Do NOT use this skill for:**
- Complex automation creation/editing (use direct API or HA UI)
- Detailed entity configuration
- Installing integrations

## How It Works

Instead of:
1. Looking up entity IDs
2. Figuring out the correct service call
3. Building JSON payloads

Just pass the user's natural language request to Home Assistant's Assist API:

```bash
curl -s -X POST "$HASS_SERVER/api/conversation/process" \
  -H "Authorization: Bearer $HASS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"text": "USER REQUEST HERE", "language": "en"}'
```

Home Assistant's built-in NLU will:
- Parse the intent
- Resolve entity names to IDs (including fuzzy matching and area awareness)
- Execute the appropriate service call
- Return a natural language response

## Setup

Set these environment variables (in OpenClaw config under `env`):

```json
{
  "env": {
    "HASS_SERVER": "https://your-homeassistant-url",
    "HASS_TOKEN": "your-long-lived-access-token"
  }
}
```

Generate a token in Home Assistant: Profile → Long-Lived Access Tokens → Create Token

## API Reference

### Endpoint

```
POST /api/conversation/process
```

### Request

```json
{
  "text": "turn on the kitchen lights",
  "language": "en"
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `text` | string | Yes | Natural language command or query |
| `language` | string | No | Language code (default: "en") |
| `conversation_id` | string | No | For multi-turn conversations |

### Response

```json
{
  "response": {
    "speech": {
      "plain": {
        "speech": "Turned on the light"
      }
    },
    "response_type": "action_done",
    "data": {
      "targets": [],
      "success": [
        {"name": "Kitchen Light", "type": "entity", "id": "light.kitchen"}
      ],
      "failed": []
    }
  },
  "conversation_id": "...",
  "continue_conversation": false
}
```

### Response Types

| Type | Meaning |
|------|---------|
| `action_done` | Command executed successfully |
| `query_answer` | Question answered (check `speech` for answer) |
| `error` | Something went wrong |

## Examples

### Turn off a light
```bash
curl -s -X POST "$HASS_SERVER/api/conversation/process" \
  -H "Authorization: Bearer $HASS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"text": "turn off the kitchen light", "language": "en"}'
```

### Query device state
```bash
curl -s -X POST "$HASS_SERVER/api/conversation/process" \
  -H "Authorization: Bearer $HASS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"text": "what lights are on?", "language": "en"}'
```

### Control by area
```bash
curl -s -X POST "$HASS_SERVER/api/conversation/process" \
  -H "Authorization: Bearer $HASS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"text": "turn off all lights in the bedroom", "language": "en"}'
```

### Set specific values
```bash
curl -s -X POST "$HASS_SERVER/api/conversation/process" \
  -H "Authorization: Bearer $HASS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"text": "set the thermostat to 70 degrees", "language": "en"}'
```

## Tips

- **Use natural names**: Say "kitchen light" not "light.kitchen_light_1" — HA resolves names
- **Areas work**: "turn off the bedroom" affects all devices in that area
- **Check the response**: The `speech.plain.speech` field contains a human-readable result
- **Failures are reported**: Check `data.failed` array if something didn't work
- **Works with any integration**: Anything exposed to HA works — Hue, Z-Wave, Zigbee, Matter, WiFi devices, etc.

## Why This Approach?

| Approach | Tokens Used | Speed | Reliability |
|----------|-------------|-------|-------------|
| Manual entity lookup + service call | High | Slow | Depends on AI's HA knowledge |
| **Assist API (this skill)** | Low | Fast | Uses HA's built-in NLU |

The Assist API leverages Home Assistant's understanding of your home — areas, device names, aliases, and entity relationships. It's what powers HA's voice assistants internally.

## Troubleshooting

**"Sorry, I couldn't understand that"**
- Check that the entity/area name matches what's in Home Assistant
- Try using the exact name shown in the HA dashboard
- Ensure the integration supports the requested action

**Empty response or error**
- Verify `HASS_SERVER` and `HASS_TOKEN` are set correctly
- Check that the Conversation integration is enabled in HA
- Test the token with a simple API call first

## Links

- [Home Assistant Conversation API Docs](https://developers.home-assistant.io/docs/intent_conversation_api/)
- [Home Assistant REST API](https://developers.home-assistant.io/docs/api/rest/)
