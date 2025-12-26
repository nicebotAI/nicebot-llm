# NiceBot LLM

A personal AI assistant API endpoint powered by a local LLM running on NVIDIA Jetson hardware. Designed for research, document creation, and conversational AI tasks.

## Overview

NiceBot LLM provides a self-hosted AI endpoint that can:
- Answer research questions and synthesize information
- Assist with document creation and writing tasks
- Integrate with automation tools like n8n for agentic workflows
- Operate entirely on your local network with no cloud dependencies

## Hardware Platform

### NVIDIA Jetson Orin Nano Super Developer Kit

| Component | Specification |
|-----------|---------------|
| **Device** | NVIDIA Jetson Orin Nano Super Developer Kit |
| **CPU** | 6-core ARM Cortex-A78AE |
| **GPU** | 1024 CUDA cores (Ampere architecture) |
| **Memory** | 8GB LPDDR5 (shared CPU/GPU) |
| **Storage** | 256GB NVMe SSD |
| **JetPack** | 6.2.1 |
| **L4T** | R36.4.7 |
| **Kernel** | 5.15.148-tegra |

### Current Configuration

```
Model: NVIDIA Jetson Orin Nano Engineering Reference Developer Kit Super
OS: Ubuntu 22.04 (aarch64)
Storage: 233GB NVMe SSD (179GB available)
Network: 10.0.0.240
```

## LLM Configuration

### Model: Gemma 3 4B

| Property | Value |
|----------|-------|
| Model | `gemma3:4b` |
| Size | 3.3 GB |
| Context Window | 96K tokens |
| Provider | Google DeepMind |

Selected for optimal balance of quality and performance on 8GB hardware. See [model comparison research](https://github.com/nicebotAI/local-model/blob/main/model_comparison.md) for details.

### Ollama Runtime

- **Version:** 0.13.5
- **Endpoint:** `http://10.0.0.240:11434`
- **API:** OpenAI-compatible

## API Usage

### Endpoints

```bash
# Generate completion
POST http://10.0.0.240:11434/api/generate

# Chat completion
POST http://10.0.0.240:11434/api/chat

# List models
GET http://10.0.0.240:11434/api/tags
```

### Example: Chat Request

```bash
curl http://10.0.0.240:11434/api/chat -d '{
  "model": "gemma3:4b",
  "messages": [
    {"role": "user", "content": "What are the key benefits of edge AI?"}
  ],
  "stream": false
}'
```

### Example: Generate Request

```bash
curl http://10.0.0.240:11434/api/generate -d '{
  "model": "gemma3:4b",
  "prompt": "Write a brief summary of quantum computing",
  "stream": false
}'
```

## n8n Integration

This endpoint integrates with n8n for building AI-powered automation workflows.

### Setup in n8n

1. Add Ollama credentials:
   - Base URL: `http://10.0.0.240:11434`

2. Use AI Agent nodes with tools:
   - Web search for research
   - HTTP requests for data gathering
   - Document generation

### Example Workflow

```
[Webhook Trigger] → [AI Agent (Gemma 3)] → [Web Search] → [Format Response] → [Output]
```

## Project Structure

```
nicebot-llm/
├── README.md           # This file
├── LICENSE             # MIT License
├── .gitignore          # Python gitignore
├── src/                # Source code (future)
├── docs/               # Documentation (future)
└── examples/           # API usage examples (future)
```

## Roadmap

- [ ] Custom system prompts for different use cases
- [ ] Document RAG (Retrieval-Augmented Generation)
- [ ] Web search integration
- [ ] Conversation memory/history
- [ ] API authentication layer
- [ ] Response caching

## Related Resources

- [Local Model Repository](https://github.com/nicebotAI/local-model) - Jetson setup and configuration
- [Ollama Documentation](https://ollama.com)
- [n8n Documentation](https://docs.n8n.io/)

## License

MIT License - see [LICENSE](LICENSE) file for details.
