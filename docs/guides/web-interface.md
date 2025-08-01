---
title: "Web Interface Guide"
layout: single
permalink: /guides/web-interface/
sidebar:
  nav: "docs"
toc: true
toc_label: "Web Interface"
toc_icon: "globe"
---

## API Integration

### Using the REST API

You can interact with Insight Ingenious through its REST API:

```bash
# Start a conversation
curl -X POST http://localhost:80/api/v1/chat \
  -H "Content-Type: application/json" \
  -H "Authorization: Basic $(echo -n username:password | base64)" \
  -d '{
    "user_prompt": "Your message here",
    "conversation_flow": "your_conversation_flow"
  }'
```

### Creating Custom API Routes

1. Create a new route module in `ingenious_extensions_template/api/routes/custom.py`
2. Implement the `Api_Routes` class

Example:
```python
from fastapi import APIRouter, Depends, FastAPI
from ingenious.models.api_routes import IApiRoutes
from ingenious.models.config import Config

class Api_Routes(IApiRoutes):
    def __init__(self, config: Config, app: FastAPI):
        self.config = config
        self.app = app
        self.router = APIRouter()

    def add_custom_routes(self):
        @self.router.get("/api/v1/custom-endpoint")
        async def custom_endpoint():
            return {"message": "Custom endpoint response"}

        self.app.include_router(self.router)
```
