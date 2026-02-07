---
name: homeassistant-assist
description: Control Home Assistant smart home devices using the Assist (Conversation) API. Use this skill when the user wants to control smart home entities - lights, switches, thermostats, covers, vacuums, media players, or any other smart device. Passes natural language directly to Home Assistant's built-in NLU for fast, token-efficient control.
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

Use this skill when the user's intent is to **control or query any smart home device**. This includes anything integrated with Home Assistant — not just lights and switches, but any entity HA exposes.

Examples:
- "turn off the kitchen light"
- "set thermostat to 72"
- "close the garage door"
- "start the vacuum"
- "what's the temperature outside?"
- "is the front door locked?"

If it's in Home Assistant, this skill can control or query it.

**Do NOT use this skill for:**
- Complex automation creation/editing (use direct API or HA UI)
- Detailed entity configuration
- Installing integrations

## How It Works

Pass the user's natural language request directly to Home Assistant:

```bash
curl -s -X POST "$HASS_SERVER/api/conversation/process" \
  -H "Authorization: Bearer $HASS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"text": "USER REQUEST HERE", "language": "en"}'
```

Home Assistant's built-in NLU handles:
- Intent parsing
- Entity name resolution (fuzzy matching, area awareness)
- Service call execution
- Response generation

## Handling Responses

### For Actions (`response_type: "action_done"`)

Check `data.success[]` and `data.failed[]` to confirm what happened:

```json
{
  "response_type": "action_done",
  "data": {
    "success": [{"name": "Kitchen Light", "id": "light.kitchen"}],
    "failed": []
  }
}
```

Report based on the success/failed arrays, not just the `speech` field.

### For Queries (`response_type: "query_answer"`)

**Parse `data.success[]` for robust answers** — don't rely solely on `speech`.

The `speech` field may be vague (e.g., "Lamp and Lamp" when two devices share a name). Instead, parse the entity data:

```json
{
  "response_type": "query_answer",
  "speech": {"plain": {"speech": "Lamp and Lamp"}},
  "data": {
    "success": [
      {"name": "Lamp", "id": "light.livingroom_lamp"},
      {"name": "Lamp", "id": "light.bedroom_lamp"}
    ]
  }
}
```

From this, derive a better answer:
> "The living room lamp and bedroom lamp are on."

Extract location/context from entity IDs when friendly names are ambiguous.

### For Errors (`response_type: "error"`)

Check `data.code`:
- `no_intent_match` — HA didn't understand the request
- `no_valid_targets` — Entity/area doesn't exist
- `failed_to_handle` — Unexpected error

## Setup

Set environment variables in OpenClaw config:

```json
{
  "env": {
    "HASS_SERVER": "https://your-homeassistant-url",
    "HASS_TOKEN": "your-long-lived-access-token"
  }
}
```

Generate a token: Home Assistant → Profile → Long-Lived Access Tokens → Create Token

## API Reference

### Endpoint

```
POST /api/conversation/process
```

**Note:** Use `/api/conversation/process`, NOT `/api/services/conversation/process`. The service endpoint doesn't return the full response.

### Request

```json
{
  "text": "turn on the kitchen lights",
  "language": "en"
}
```

### Response

```json
{
  "response": {
    "speech": {
      "plain": {"speech": "Turned on the light"}
    },
    "response_type": "action_done",
    "data": {
      "targets": [],
      "success": [{"name": "Kitchen Light", "type": "entity", "id": "light.kitchen"}],
      "failed": []
    }
  },
  "conversation_id": "...",
  "continue_conversation": false
}
```

## Tips

- **Natural names work**: Say "kitchen light" not "light.kitchen_light_1"
- **Areas work**: "turn off the bedroom" affects all devices in that area
- **Any device type**: If HA can control it, this API can too
- **Parse entity IDs**: When names are ambiguous, extract context from the entity ID (e.g., `light.bedroom_lamp` → bedroom)

## Why This Approach?

| Approach | Tokens | Speed | Reliability |
|----------|--------|-------|-------------|
| Manual entity lookup + service calls | High | Slow | Depends on AI's HA knowledge |
| **Assist API** | Low | Fast | Uses HA's built-in NLU |

The Assist API leverages Home Assistant's understanding of your home — areas, device names, aliases, and entity relationships.

## Links

- [Home Assistant Conversation API Docs](https://developers.home-assistant.io/docs/intent_conversation_api/)
- [Home Assistant REST API](https://developers.home-assistant.io/docs/api/rest/)
