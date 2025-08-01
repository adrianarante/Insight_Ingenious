---
title: "Quick Onboarding"
layout: single
permalink: /quick_onboarding/
sidebar:
  nav: "docs"
toc: true
toc_label: "Quick Onboarding"
toc_icon: "rocket"
---

This guide provides a fast-track introduction to Insight Ingenious for new users who want to get up and running quickly.

## What is Insight Ingenious?

Insight Ingenious is an enterprise-grade Python library for building AI agent APIs with Microsoft Azure integrations. It provides:

- Multi-agent conversation workflows
- Azure OpenAI integration
- Configurable chat history storage
- Document processing capabilities
- Extensible architecture

## Prerequisites

- Python 3.13 or higher
- `uv` package manager
- Azure OpenAI API credentials

## Quick Start Steps

1. **Initialize new uv project and install Ingenious**
   ```bash
   uv init
   uv add ingenious
   ```

2. **Initialize a new Ingenious project**
   ```bash
   uv run ingen init
   ```

3. **Configure your environment**
   Create a `.env` file with your Azure OpenAI credentials:
   ```env
   INGENIOUS_MODELS__0__API_KEY=your-api-key
   INGENIOUS_MODELS__0__BASE_URL=https://your-resource.openai.azure.com/
   INGENIOUS_MODELS__0__MODEL=gpt-4o-mini
   INGENIOUS_CHAT_SERVICE__TYPE=multi_agent
   ```

4. **Start the server**
   ```bash
   uv run ingen serve
   ```

5. **Test the installation**
   ```bash
   curl -X POST http://localhost:8000/api/v1/chat \
     -H "Content-Type: application/json" \
     -d '{
       "user_prompt": "Hello, world!",
       "conversation_flow": "classification-agent"
     }'
   ```

## Next Steps

- [Detailed Getting Started Guide](../getting-started/README.md)
- [Configuration Guide](../getting-started/configuration.md)
- [API Integration Guide](../guides/api-integration.md)
- [Development Guide](../development/README.md)

## Common Tasks

### Running a workflow
```bash
uv run ingen workflows
```

### Checking system status
```bash
uv run ingen status
```

### Viewing logs
```bash
tail -f server.log
```

## Getting Help

- [Troubleshooting Guide](../troubleshooting/README.md)
- [CLI Reference](../CLI_REFERENCE.md)
- [API Documentation](../api/README.md)
