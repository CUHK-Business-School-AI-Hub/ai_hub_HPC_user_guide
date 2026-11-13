# Local LLMs

The AI Hub hosts two local large language models, which you can access through an OpenAI-compatible API or a WebUI from within the cluster network.

## Available Models

| Model | API Endpoint | Description |
| :--- | :--- | :--- |
| Qwen3.8-27B | `http://137.189.75.113:8000/qwen/v1/chat/completions` | General-purpose text LLM |
| Deepseek-VL2-Small | `http://137.189.75.113:8000/deepseek-vl2-small/v1/chat/completions` | Vision-language model; supports image input |

## Access via API

### Qwen3.8-27B

Example request with `curl`:

```bash
curl http://137.189.75.113:8000/qwen/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer your-secret-key" \
  -d '{
    "model": "qwen3.8-27b",
    "messages": [
      {"role": "system", "content": "你是一个高效的 AI 助手。"},
      {"role": "user", "content": "请用一句话介绍你自己。"}
    ],
    "temperature": 0.7,
    "max_tokens": 1024
  }'
```

The `Authorization` header is not validated; you can fill in any string as its value.

### Deepseek-VL2-Small

Deepseek-VL2-Small accepts image inputs. Convert the image to base64 encoding first, then pass it in the `image_url` field with a `data:image/jpeg;base64,` prefix, alongside the text question:

```bash
curl http://137.189.75.113:8000/deepseek-vl2-small/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "deepseek-vl2-small",
    "messages": [
      {
        "role": "user",
        "content": [
          {"type": "text", "text": "这张图片里有什么？"},
          {
            "type": "image_url",
            "image_url": {
              "url": "data:image/jpeg;base64,<BASE64_ENCODED_IMAGE>"
            }
          }
        ]
      }
    ],
    "temperature": 0.7,
    "max_tokens": 512
  }'
```

Replace `<BASE64_ENCODED_IMAGE>` with the actual base64 content of your image. You can encode an image as a single line of base64 with:

```bash
base64 -i image.jpg | tr -d '\n'
```

or in Python:

```python
import base64

with open("image.jpg", "rb") as f:
    encoded = base64.b64encode(f.read()).decode("utf-8")
```

## Access via WebUI

A WebUI is also available at `http://137.189.75.113`.

On your first visit, you need to register a new account. If your account requires approval, please email Jie Ke at `bafhpc@cuhk.edu.hk`.

## Using Claude Code in VS Code

You can run [Claude Code](https://code.claude.com/docs), Anthropic's coding agent, inside VS Code and point it at the cluster's local Qwen3.8-27B model through the Anthropic-compatible endpoint.

> 💡 Note: The endpoint is only reachable from within the cluster network. If you connect to the cluster from VS Code using Remote-SSH (see the main handbook), install and configure the extension on the remote side so that it can reach the endpoint.

### Install and configure the extension

1. In VS Code, open the Extensions view (`Ctrl+Shift+X` / `Cmd+Shift+X`), search for `Claude Code`, and install the official extension (publisher: Anthropic). VS Code 1.94 or later is required.

2. Open the settings JSON (`Ctrl+Shift+P` / `Cmd+Shift+P` → `Preferences: Open User Settings (JSON)`) and add the following:

   ```json
   {
     "claudeCode.environmentVariables": [
       {"name": "ANTHROPIC_BASE_URL", "value": "http://137.189.75.113:8000/qwen"},
       {"name": "ANTHROPIC_AUTH_TOKEN", "value": "your-secret-key"},
       {"name": "ANTHROPIC_MODEL", "value": "qwen3.8-27b"},
       {"name": "ANTHROPIC_DEFAULT_OPUS_MODEL", "value": "qwen3.8-27b"},
       {"name": "ANTHROPIC_DEFAULT_SONNET_MODEL", "value": "qwen3.8-27b"},
       {"name": "ANTHROPIC_DEFAULT_HAIKU_MODEL", "value": "qwen3.8-27b"},
       {"name": "CLAUDE_CODE_EFFORT_LEVEL", "value": "xhigh"}
     ]
   }
   ```

   The variables are:

   | Variable | Purpose |
   | :--- | :--- |
   | `ANTHROPIC_BASE_URL` | The Anthropic-compatible endpoint of the local model. Note that, unlike the OpenAI-compatible URL above, it has no `/v1` suffix. |
   | `ANTHROPIC_AUTH_TOKEN` | The API key sent with each request. It is not validated by the local server, so any string works. |
   | `ANTHROPIC_MODEL` | The model used for the main conversation. |
   | `ANTHROPIC_DEFAULT_OPUS_MODEL`<br>`ANTHROPIC_DEFAULT_SONNET_MODEL`<br>`ANTHROPIC_DEFAULT_HAIKU_MODEL` | Claude Code internally routes different tasks (main reasoning, quick lookups, summarisation) to different model tiers. Pointing all three at `qwen3.8-27b` ensures every task runs on the local model. |
   | `CLAUDE_CODE_EFFORT_LEVEL` | Optional reasoning-effort level: `low`, `medium`, `high`, or `xhigh`. |

3. Since the endpoint is a third-party provider, skip the claude.ai sign-in prompt by adding:

   ```json
   "claudeCode.disableLoginPrompt": true
   ```

4. Reload the VS Code window (`Developer: Reload Window`) and open the Claude Code panel from the spark icon in the editor toolbar or activity bar. You can now start chatting as usual; all requests are sent to the local model.

### Using Claude Code from the terminal (optional)

If you prefer the CLI, put the same variables in `~/.claude/settings.json` on the machine that runs Claude Code (e.g., the cluster management node). Settings there are shared between the CLI and the VS Code extension:

```json
{
  "env": {
    "ANTHROPIC_BASE_URL": "http://137.189.75.113:8000/qwen",
    "ANTHROPIC_AUTH_TOKEN": "your-secret-key",
    "ANTHROPIC_MODEL": "qwen3.8-27b",
    "ANTHROPIC_DEFAULT_OPUS_MODEL": "qwen3.8-27b",
    "ANTHROPIC_DEFAULT_SONNET_MODEL": "qwen3.8-27b",
    "ANTHROPIC_DEFAULT_HAIKU_MODEL": "qwen3.8-27b"
  }
}
```

Then run `claude` in the terminal.
