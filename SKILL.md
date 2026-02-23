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

Control smart home devices by passing natural language to Home Assistant's Assist (Conversation) API. **Fire and forget** — trust Assist to handle intent parsing, entity resolution, and execution.

## When to Use This Skill

Use this skill when the user wants to **control or query any smart home device**. If it's in Home Assistant, Assist can handle it.

## How It Works

Pass the user's request directly to Assist:

```bash
AGENT_ID="${HASS_AGENT_ID:-home_assistant}"
curl -s -X POST "$HASS_SERVER/api/conversation/process" \
  -H "Authorization: Bearer $HASS_TOKEN" \
  -H "Content-Type: application/json" \
  -d "{\"text\": \"USER REQUEST HERE\", \"language\": \"en\", \"agent_id\": \"$AGENT_ID\"}"
```

**Trust Assist.** It handles:
- Intent parsing
- Fuzzy entity name matching
- Area-aware commands
- Execution
- Error responses

## Handling Responses

**Just relay what Assist says.** The `response.speech.plain.speech` field contains the human-readable result.

- `"Turned on the light"` → Success, tell the user
- `"Sorry, I couldn't understand that"` → Assist couldn't parse it
- `"Sorry, there are multiple devices called X"` → Ambiguous name

**Don't over-interpret.** If Assist says it worked, it worked. Trust the response.

## When Assist Returns an Error

Only if Assist returns an error (`response_type: "error"`), you can **suggest HA-side improvements**:

| Error | Suggestion |
|-------|------------|
| `no_intent_match` | "HA didn't recognize that command" |
| `no_valid_targets` | "Try checking the entity name in HA, or add an alias" |
| Multiple devices | "There may be duplicate names — consider adding unique aliases in HA" |

These are **suggestions for improving HA config**, not skill failures. The skill did its job — it passed the request to Assist.

## Setup

Set environment variables in OpenClaw config:

```json
{
  "env": {
    "HASS_SERVER": "https://your-homeassistant-url",
    "HASS_TOKEN": "your-long-lived-access-token",
    "HASS_AGENT_ID": "home_assistant"
  }
}
```

**Required:**
- `HASS_SERVER` - Your Home Assistant URL
- `HASS_TOKEN` - Long-lived access token

**Optional:**
- `HASS_AGENT_ID` - Conversation agent to use (defaults to `home_assistant`)

Generate a token: Home Assistant → Profile → Long-Lived Access Tokens → Create Token

### Using Different Conversation Agents

By default, requests go to Home Assistant's built-in Assist (`home_assistant`). You can route to other agents:

- `home_assistant` - Local Assist (default, fast, private)
- `conversation.google_generative_ai` - Google Gemini
- `conversation.chatgpt` - OpenAI ChatGPT
- Custom agents you've configured in HA

**Example:** Route general questions to ChatGPT while keeping smart home commands local:

```json
{
  "env": {
    "HASS_SERVER": "https://your-homeassistant-url",
    "HASS_TOKEN": "your-token",
    "HASS_AGENT_ID": "conversation.chatgpt"
  }
}
```

To find your agent IDs: Home Assistant → Settings → Voice assistants → Your agent → Copy the entity ID

## API Reference

### Endpoint

```
POST /api/conversation/process
```

**Note:** Use `/api/conversation/process`, NOT `/api/services/conversation/process`.

### Request

```json
{
  "text": "turn on the kitchen lights",
  "language": "en",
  "agent_id": "home_assistant"
}
```

**Fields:**
- `text` (required) - The command or question
- `language` (optional) - Defaults to `"en"`
- `agent_id` (optional) - Defaults to `"home_assistant"`

### Response

```json
{
  "response": {
    "speech": {
      "plain": {"speech": "Turned on the light"}
    },
    "response_type": "action_done",
    "data": {
      "success": [{"name": "Kitchen Light", "id": "light.kitchen"}],
      "failed": []
    }
  }
}
```

## Philosophy

- **Trust Assist** — It knows the user's HA setup better than we do
- **Fire and forget** — Pass the request, relay the response
- **Don't troubleshoot** — If something doesn't work, suggest HA config improvements
- **Keep it simple** — One API call, natural language in, natural language out

## Links

- [Home Assistant Conversation API Docs](https://developers.home-assistant.io/docs/intent_conversation_api/)
