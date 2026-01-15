# Smart Webhook Router

An n8n workflow that receives incoming webhooks, validates input, routes requests by priority level, and returns structured JSON responses.

## Features

- **Webhook Endpoint**: Accepts POST requests at `/webhook/smart-router`
- **Input Validation**: Validates that required `event` field is present
- **Priority Routing**: Routes to different processing paths based on priority (high/medium/low)
- **Structured Responses**: Returns consistent JSON responses with processing metadata
- **Error Handling**: Returns 400 errors for invalid requests

## Workflow Architecture

```
Webhook Trigger -> Validate Input -> Route by Priority -> Process -> Merge -> Response
                         |
                         v
                   Error Response (400)
```

## Request Format

```json
{
  "event": "user.created",
  "priority": "high",
  "data": {
    "userId": "123",
    "email": "user@example.com"
  }
}
```

### Fields

| Field | Required | Description |
|-------|----------|-------------|
| `event` | Yes | Event type identifier |
| `priority` | No | Priority level: `high`, `medium`, or `low` (defaults to `low`) |
| `data` | No | Additional event payload |

## Response Format

### Success (200)

```json
{
  "success": true,
  "data": {
    "status": "processed",
    "priority_level": "HIGH",
    "event": "user.created",
    "processed_at": "2026-01-15T08:57:00.000Z",
    "action": "immediate_notification"
  },
  "message": "Event processed successfully"
}
```

### Error (400)

```json
{
  "success": false,
  "error": "Invalid request: missing required field \"event\""
}
```

## Priority Actions

| Priority | Action |
|----------|--------|
| `high` | `immediate_notification` |
| `medium` | `queue_for_review` |
| `low` | `log_only` |

## Installation

1. Open your n8n instance
2. Go to **Workflows** > **Import from File**
3. Select `workflow.json` from this repository
4. Activate the workflow

## Testing

Once activated, test with curl:

```bash
# High priority event
curl -X POST http://localhost:5678/webhook/smart-router \
  -H "Content-Type: application/json" \
  -d '{"event": "alert.triggered", "priority": "high"}'

# Invalid request (missing event)
curl -X POST http://localhost:5678/webhook/smart-router \
  -H "Content-Type: application/json" \
  -d '{"priority": "high"}'
```

## Customization

You can extend this workflow by:

- Adding more priority levels in the Switch node
- Connecting external services (Slack, email) to the processing nodes
- Adding database logging after the Merge node
- Implementing rate limiting at the webhook level

## License

MIT
