# @comet-ml/openclaw-opik

Export OpenClaw agent traces to [Opik](https://www.comet.com/docs/opik/) for
LLM observability — see prompts, completions, tool calls, token usage, and
costs in the Opik UI.

The Opik plugin runs **inside the Gateway process**. If you use a remote
Gateway, install and configure the plugin on that machine, then restart the
Gateway to load it.

## Setup

### Interactive wizard (recommended)

```bash
openclaw opik configure
```

The wizard validates your URL and API key, auto-detects local instances, and
writes the config for you. Restart the Gateway afterwards.

### CLI config commands

```bash
openclaw config set plugins.entries.opik.enabled true
openclaw config set plugins.entries.opik.config.apiKey "your-api-key"
openclaw config set plugins.entries.opik.config.projectName "my-openclaw"
```

### Manual config

Add to your `~/.openclaw/config.json`:

```json
{
  "plugins": {
    "entries": {
      "opik": {
        "enabled": true,
        "config": {
          "enabled": true,
          "apiKey": "your-api-key",
          "projectName": "my-openclaw"
        }
      }
    }
  }
}
```

### Self-hosted / local Opik

```json
{
  "plugins": {
    "entries": {
      "opik": {
        "enabled": true,
        "config": {
          "enabled": true,
          "apiUrl": "http://localhost:5173/api",
          "projectName": "openclaw-local"
        }
      }
    }
  }
}
```

## CLI commands

| Command                   | Description                     |
| ------------------------- | ------------------------------- |
| `openclaw opik configure` | Interactive setup wizard        |
| `openclaw opik status`    | Show current Opik configuration |

## Check status

```bash
openclaw opik status
```

## Verify traces

1. Start the Gateway (`openclaw gateway run`).
2. Send a message: `openclaw message send "Hello, trace me"`.
3. Look for `opik: exporting traces to project "openclaw"` in the
   Gateway log.
4. Open the Opik UI — traces appear under your project within a few seconds.

## What Gets Traced

| Event       | Opik Entity      | Data                                                                   |
| ----------- | ---------------- | ---------------------------------------------------------------------- |
| LLM call    | Trace + LLM Span | Prompt, system prompt, history, response, model, provider, token usage |
| Tool call   | Tool Span        | Tool name, input params, output/result, errors, duration               |
| Agent run   | Trace            | Duration, success/error, cost                                          |
| Model usage | Trace metadata   | Cost (USD), context window utilization                                 |

## Environment Variables

| Variable            | Description            | Default                          |
| ------------------- | ---------------------- | -------------------------------- |
| `OPIK_API_KEY`      | API key for Opik Cloud | —                                |
| `OPIK_URL_OVERRIDE` | Opik API endpoint      | `https://www.comet.com/opik/api` |
| `OPIK_PROJECT_NAME` | Project name in Opik   | `openclaw`                       |
| `OPIK_WORKSPACE`    | Workspace name         | `default`                        |

## Config Options

| Key                                    | Type       | Default                 | Description                                |
| -------------------------------------- | ---------- | ----------------------- | ------------------------------------------ |
| `plugins.entries.opik.enabled`         | `boolean`  | plugin-default          | Enable/disable plugin loading              |
| `plugins.entries.opik.config.enabled`  | `boolean`  | `true`                  | Enable/disable trace export after load     |
| `plugins.entries.opik.config.apiKey`   | `string`   | env `OPIK_API_KEY`      | API key                                    |
| `plugins.entries.opik.config.apiUrl`   | `string`   | env `OPIK_URL_OVERRIDE` | API endpoint                               |
| `plugins.entries.opik.config.projectName` | `string` | `"openclaw"`            | Project name                               |
| `plugins.entries.opik.config.workspaceName` | `string` | `"default"`           | Workspace name                             |
| `plugins.entries.opik.config.tags`     | `string[]` | `["openclaw"]`          | Default tags applied to new traces         |

## Troubleshooting

- **Traces not appearing** — Check `openclaw opik status` shows `Enabled: yes`
  and the Gateway log contains `opik: exporting traces to project "openclaw"`.
  If missing, verify `plugins.entries.opik.enabled` and
  `plugins.entries.opik.config.enabled` are both `true`, then restart the Gateway.
- **API key rejected** — Re-run `openclaw opik configure` to re-validate. For
  self-hosted instances without auth, remove
  `plugins.entries.opik.config.apiKey` from your config.
- **Self-hosted not reachable** — Verify with `curl <url>/api/health`. Local
  instances use `/api`, cloud/self-hosted use `/opik/api`. Check firewall rules
  if Gateway and Opik are on different hosts.
- **Tool spans missing/crossed in high concurrency** — Some OpenClaw embedded
  paths currently omit `sessionKey` in `after_tool_call`. This plugin uses a
  best-effort fallback (single active trace or latest active session), which can
  mis-correlate tool spans if multiple sessions are active concurrently.

For the full setup guide, see
[docs.openclaw.ai/plugins/opik](https://docs.openclaw.ai/plugins/opik).
