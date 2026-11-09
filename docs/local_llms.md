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
