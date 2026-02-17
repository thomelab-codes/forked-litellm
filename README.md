# LiteLLM (Fork) 🚅

![GitHub last commit](https://img.shields.io/github/last-commit/thomelab-codes/forked-litellm?color=red)
![GitHub repo size](https://img.shields.io/github/repo-size/thomelab-codes/forked-litellm)
![GitHub top language](https://img.shields.io/github/languages/top/thomelab-codes/forked-litellm)

> A community fork of [LiteLLM](https://github.com/BerriAI/litellm) — a unified Python SDK and Proxy Server (AI Gateway) to call 100+ LLM APIs in OpenAI format, with cost tracking, guardrails, load balancing, and logging.

This fork is maintained by [thomelab-codes](https://github.com/thomelab-codes) and includes additional features and fixes on top of the upstream project. For upstream documentation, see the [LiteLLM Docs](https://docs.litellm.ai/docs/).

---

## Table of Contents

- [Fork Changes](#fork-changes)
- [Quick Start](#quick-start)
- [Multimodal Content Support](#multimodal-content-support)
- [Supported Providers](#supported-providers)
- [Run in Developer Mode](#run-in-developer-mode)
- [Contributing](#contributing)
- [License](#license)
- [Support](#support)

---

## Fork Changes

This fork includes the following changes on top of the upstream [BerriAI/litellm](https://github.com/BerriAI/litellm):

### `video_url` Content Type Handling

The upstream `ChatCompletionVideoObject` type was defined but never handled in the OpenAI or Gemini chat transformation pipelines. Sending `{"type": "video_url", ...}` content parts would pass through untransformed, causing API errors. This fork adds proper support:

- **OpenAI** (`gpt_transformation.py`) — Added `video_url` handling in `_apply_common_transform_content_item()`, normalizing string URLs to dict format and filtering litellm-specific params (matching the existing `image_url` pattern).
- **Gemini / Vertex AI** (`vertex_ai/gemini/transformation.py`) — Added `video_url` branch in `_gemini_convert_messages_with_history()`, converting to Gemini native format via `_process_gemini_media()` with support for both string and dict URLs (GCS + HTTPS).

**Usage example:**

```python
from litellm import completion

# OpenAI with video input
response = completion(
    model="openai/gpt-4o",
    messages=[{
        "role": "user",
        "content": [
            {"type": "text", "text": "What happens in this video?"},
            {"type": "video_url", "video_url": {"url": "https://example.com/video.mp4"}}
        ]
    }]
)

# Gemini with video input (GCS or HTTPS)
response = completion(
    model="gemini/gemini-2.5-flash",
    messages=[{
        "role": "user",
        "content": [
            {"type": "text", "text": "Summarize this video."},
            {"type": "video_url", "video_url": {"url": "gs://my-bucket/video.mp4"}}
        ]
    }]
)
```

---

## Quick Start

### Python SDK

```shell
pip install litellm
```

```python
from litellm import completion
import os

os.environ["OPENAI_API_KEY"] = "your-openai-key"
os.environ["ANTHROPIC_API_KEY"] = "your-anthropic-key"

# OpenAI
response = completion(model="openai/gpt-4o", messages=[{"role": "user", "content": "Hello!"}])

# Anthropic
response = completion(model="anthropic/claude-sonnet-4-20250514", messages=[{"role": "user", "content": "Hello!"}])
```

### AI Gateway (Proxy Server)

```shell
pip install 'litellm[proxy]'
litellm --model gpt-4o
```

```python
import openai

client = openai.OpenAI(api_key="anything", base_url="http://0.0.0.0:4000")
response = client.chat.completions.create(
    model="gpt-4o",
    messages=[{"role": "user", "content": "Hello!"}]
)
```

### Docker Compose

```bash
git clone https://github.com/thomelab-codes/forked-litellm.git
cd forked-litellm
cp .env.example .env
# Edit .env to set API keys and database credentials
docker compose up -d
```

The proxy server will be available at [http://localhost:4000](http://localhost:4000).

For the full list of supported endpoints, see the [upstream docs](https://docs.litellm.ai/docs/supported_endpoints).

---

## Multimodal Content Support

LiteLLM provides a unified interface for sending multimodal content (text, images, video, audio) across providers. Use the standard OpenAI content format, and LiteLLM handles the translation to each provider's native API.

| Content Type | Format | Supported Providers |
|---|---|---|
| Text | `{"type": "text", "text": "..."}` | All providers |
| Image URL | `{"type": "image_url", "image_url": {"url": "..."}}` | OpenAI, Gemini, Vertex AI, Anthropic, Azure, Bedrock, and more |
| Video URL | `{"type": "video_url", "video_url": {"url": "..."}}` | OpenAI, Gemini, Vertex AI, Hosted VLLM |
| Audio | `{"type": "input_audio", "input_audio": {"data": "...", "format": "wav"}}` | OpenAI, Gemini, Vertex AI |
| File | `{"type": "file", "file": {"file_id": "..."}}` | OpenAI, Gemini, Vertex AI |

---

## Supported Providers

LiteLLM supports 100+ providers. For the full list with endpoint support details, see the [upstream provider docs](https://docs.litellm.ai/docs/providers) or [supported models](https://models.litellm.ai/).

<details>
<summary><b>Click to expand full provider table</b></summary>

| Provider | `/chat/completions` | `/messages` | `/responses` | `/embeddings` | `/image/generations` | `/videos` | `/audio/transcriptions` | `/audio/speech` | `/moderations` | `/batches` | `/rerank` |
|---|---|---|---|---|---|---|---|---|---|---|---|
| [Anthropic (`anthropic`)](https://docs.litellm.ai/docs/providers/anthropic) | ✅ | ✅ | ✅ |  |  | |  |  |  | ✅ |  |
| [AWS - Bedrock (`bedrock`)](https://docs.litellm.ai/docs/providers/bedrock) | ✅ | ✅ | ✅ | ✅ |  | |  |  |  |  | ✅ |
| [Azure (`azure`)](https://docs.litellm.ai/docs/providers/azure) | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |  |
| [Azure AI (`azure_ai`)](https://docs.litellm.ai/docs/providers/azure_ai) | ✅ | ✅ | ✅ | ✅ | ✅ | | ✅ | ✅ | ✅ | ✅ |  |
| [Cohere (`cohere`)](https://docs.litellm.ai/docs/providers/cohere) | ✅ | ✅ | ✅ | ✅ |  | |  |  |  |  | ✅ |
| [Databricks (`databricks`)](https://docs.litellm.ai/docs/providers/databricks) | ✅ | ✅ | ✅ |  |  | |  |  |  |  |  |
| [Deepseek (`deepseek`)](https://docs.litellm.ai/docs/providers/deepseek) | ✅ | ✅ | ✅ |  |  | |  |  |  |  |  |
| [Fireworks AI (`fireworks_ai`)](https://docs.litellm.ai/docs/providers/fireworks_ai) | ✅ | ✅ | ✅ |  |  | |  |  |  |  |  |
| [Google AI Studio - Gemini (`gemini`)](https://docs.litellm.ai/docs/providers/gemini) | ✅ | ✅ | ✅ |  |  | ✅ |  |  |  |  |  |
| [Google - Vertex AI (`vertex_ai`)](https://docs.litellm.ai/docs/providers/vertex) | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |  |  |  |  |  |
| [Groq AI (`groq`)](https://docs.litellm.ai/docs/providers/groq) | ✅ | ✅ | ✅ |  |  | |  |  |  |  |  |
| [Huggingface (`huggingface`)](https://docs.litellm.ai/docs/providers/huggingface) | ✅ | ✅ | ✅ | ✅ |  | |  |  |  |  | ✅ |
| [Mistral AI API (`mistral`)](https://docs.litellm.ai/docs/providers/mistral) | ✅ | ✅ | ✅ | ✅ |  | |  |  |  |  |  |
| [Ollama (`ollama`)](https://docs.litellm.ai/docs/providers/ollama) | ✅ | ✅ | ✅ | ✅ |  | |  |  |  |  |  |
| [OpenAI (`openai`)](https://docs.litellm.ai/docs/providers/openai) | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |  |
| [OpenRouter (`openrouter`)](https://docs.litellm.ai/docs/providers/openrouter) | ✅ | ✅ | ✅ |  |  | |  |  |  |  |  |
| [Perplexity AI (`perplexity`)](https://docs.litellm.ai/docs/providers/perplexity) | ✅ | ✅ | ✅ |  |  | |  |  |  |  |  |
| [Together AI (`together_ai`)](https://docs.litellm.ai/docs/providers/togetherai) | ✅ | ✅ | ✅ |  |  | |  |  |  |  |  |
| [xAI (`xai`)](https://docs.litellm.ai/docs/providers/xai) | ✅ | ✅ | ✅ |  |  | |  |  |  |  |  |
| ... and [80+ more](https://docs.litellm.ai/docs/providers) |  |  |  |  |  | |  |  |  |  |  |

</details>

---

## Run in Developer Mode

### Services

1. Setup `.env` file in root (`cp .env.example .env`)
2. Run dependent services: `docker compose up db prometheus`

### Backend

1. Create virtual environment: `python -m venv .venv`
2. Activate virtual environment: `source .venv/bin/activate`
3. Install dependencies: `pip install -e ".[all]"`
4. Install Prisma: `pip install prisma`
5. Generate Prisma client: `prisma generate`
6. Start proxy backend: `python litellm/proxy/proxy_cli.py`

### Frontend

1. Navigate to `ui/litellm-dashboard`
2. Install dependencies: `npm install`
3. Run `npm run dev` to start the dashboard

---

## Contributing

We welcome contributions to this fork! Whether you're fixing bugs, adding features, or improving documentation, we appreciate your help.

```bash
git clone https://github.com/thomelab-codes/forked-litellm.git
cd forked-litellm
make install-dev    # Install development dependencies
make format         # Format your code
make lint           # Run all linting checks
make test-unit      # Run unit tests
```

For detailed contributing guidelines, see [CONTRIBUTING.md](CONTRIBUTING.md).

### Code Quality

LiteLLM follows the [Google Python Style Guide](https://google.github.io/styleguide/pyguide.html). Our automated checks include **Black** (formatting), **Ruff** (linting), **MyPy** (type checking), circular import detection, and import safety checks.

---

## License

This project is licensed under the MIT License (with enterprise components under a separate license). See the [LICENSE](./LICENSE) file for details.

---

## Support

If you have questions, suggestions, or need assistance with this fork, please [open an issue](https://github.com/thomelab-codes/forked-litellm/issues) in this repository.

For upstream community support:
- [LiteLLM Documentation](https://docs.litellm.ai/docs/)
- [Community Discord](https://discord.gg/wuPM9dRgDw)

---

> Originally created by [BerriAI](https://github.com/BerriAI). Fork maintained by [thomelab-codes](https://github.com/thomelab-codes).
