---
title: "API Reference"
layout: single
permalink: /api/
sidebar:
  nav: "docs"
toc: true
toc_label: "API Sections"
toc_icon: "plug"
---

Complete API documentation for Insight Ingenious - an enterprise-grade Python library for quickly setting up APIs to interact with AI Agents. This reference covers REST endpoints, workflow examples, debugging utilities, and integration guides.

## API Architecture Overview

```mermaid
flowchart TB
 subgraph subGraph0["Client Applications"]
        WEB_CLIENT["Web Applications"]
        MOBILE_CLIENT["Mobile Apps"]
        API_CLIENT["API Integrations"]
        CLI_CLIENT["CLI Tools"]
  end
 subgraph subGraph1["API Layer"]
        FASTAPI["FastAPI Application"]
        CHAT_API["Chat API - <br>/api/v1/chat"]
        DIAGNOSTIC_API["Diagnostic API - /api/v1/workflow-status/{workflow_name}"]
        PROMPTS_API["Prompts API -<br>/api/v1/prompts"]
        FEEDBACK_API["Feedback API - /api/v1/messages/{message_id}/feedback"]
        AUTH_API["Auth API - <br>/api/v1/auth"]
  end
 subgraph subGraph2["Backend Services"]
        CHAT_SERVICE["Chat Service"]
        MULTI_AGENT_SERVICE["Multi-Agent Service"]
        CONFIG_SERVICE["Config Service"]
        FILE_STORAGE["File Storage"]
  end
 subgraph subGraph3["External Services"]
        AZURE_OPENAI["Azure OpenAI"]
        AZURE_SEARCH["Azure Search"]
        AZURE_SQL["Azure SQL"]
  end
    WEB_CLIENT --> FASTAPI
    MOBILE_CLIENT --> FASTAPI
    API_CLIENT --> FASTAPI
    CLI_CLIENT --> FASTAPI
    FASTAPI --> CHAT_API & DIAGNOSTIC_API & PROMPTS_API & FEEDBACK_API & AUTH_API
    CHAT_API --> CHAT_SERVICE
    DIAGNOSTIC_API --> CONFIG_SERVICE
    PROMPTS_API --> FILE_STORAGE
    FEEDBACK_API --> CHAT_SERVICE
    AUTH_API --> CHAT_SERVICE
    CHAT_SERVICE --> MULTI_AGENT_SERVICE
    MULTI_AGENT_SERVICE --> FILE_STORAGE & AZURE_OPENAI
    CONFIG_SERVICE --> AZURE_SEARCH & AZURE_SQL

     WEB_CLIENT:::client
     MOBILE_CLIENT:::client
     API_CLIENT:::client
     CLI_CLIENT:::client
     FASTAPI:::api
     CHAT_API:::api
     DIAGNOSTIC_API:::api
     PROMPTS_API:::api
     FEEDBACK_API:::api
     AUTH_API:::api
     CHAT_SERVICE:::service
     MULTI_AGENT_SERVICE:::service
     CONFIG_SERVICE:::service
     FILE_STORAGE:::service
     AZURE_OPENAI:::external
     AZURE_SEARCH:::external
     AZURE_SQL:::external
    classDef client fill:#e8f5e8
    classDef api fill:#fff3e0
    classDef service fill:#e3f2fd
    classDef external fill:#fce4ec
```

## API Request Flow

```mermaid
sequenceDiagram
    participant Client
    participant Gateway as API Gateway
    participant Auth as Auth Service
    participant API as API Endpoint
    participant Service as Backend Service
    participant LLM as Azure OpenAI
    participant Storage as Storage

    Client->>Gateway: HTTP Request
    Gateway->>Auth: Validate API Key
    Auth-->>Gateway: Authentication Success

    Gateway->>API: Route Request
    API->>Service: Process Request

    Service->>Storage: Load Context
    Storage-->>Service: Previous History

    Service->>LLM: Generate Response
    LLM-->>Service: AI Response

    Service->>Storage: Save Response
    Service-->>API: Formatted Response

    API-->>Gateway: HTTP Response
    Gateway-->>Client: JSON Response

    Note over Client,Storage: Error handling at each step with appropriate HTTP status codes
```

## Core API Endpoints

### Endpoint Overview

```mermaid
graph LR
    subgraph "Chat Endpoints"
        CHAT_POST[POST <br>/api/v1/chat<br>Send Message]
    end

    subgraph "Diagnostic Endpoints"
        WORKFLOW_STATUS["GET <br>/api/v1/workflow-status/{workflow_name}<br>Check Workflow Status"]
        WORKFLOWS_LIST[GET <br>/api/v1/workflows<br>List All Workflows]
        DIAGNOSTIC[GET <br>/api/v1/diagnostic<br>System Diagnostic]
    end

    subgraph "System Endpoints"
        HEALTH[GET <br>/api/v1/health<br>Health Check]
    end

    subgraph "Management Endpoints"
        PROMPTS_VIEW["GET <br>/api/v1/prompts/view/{revision_id}/{filename}<br>View Prompt"]
        PROMPTS_LIST["GET <br>/api/v1/prompts/list/{revision_id}<br>List Prompts"]
        PROMPTS_UPDATE["POST <br>/api/v1/prompts/update/{revision_id}/{filename}<br>Update Prompt"]
        PROMPTS_REVISIONS[GET <br>/api/v1/revisions/list <br>List All Revisions]
        PROMPTS_WORKFLOWS["GET <br>/api/v1/workflows/list<br>List Workflows with Prompts"]
        FEEDBACK["PUT <br>/api/v1/messages/{message_id}/feedback<br>Message Feedback"]
        CONVERSATIONS["GET /api/v1/conversations/{thread_id}<br>Get Conversation History"]
    end

    subgraph "Authentication Endpoints"
        AUTH_LOGIN[POST <br>/api/v1/auth/login <br>JWT Login]
        AUTH_REFRESH[POST <br>/api/v1/auth/refresh<br>Refresh Token]
        AUTH_VERIFY[GET <br>/api/v1/auth/verify<br>Verify Token]
    end

    classDef chat fill:#e8f5e8
    classDef workflow fill:#fff3e0
    classDef system fill:#e3f2fd
    classDef management fill:#f3e5f5
    classDef auth fill:#ffe0b2

    class CHAT_POST chat
    class WORKFLOW_STATUS,WORKFLOWS_LIST,DIAGNOSTIC workflow
    class HEALTH system
    class PROMPTS_VIEW,PROMPTS_LIST,PROMPTS_UPDATE,PROMPTS_REVISIONS,PROMPTS_WORKFLOWS,FEEDBACK,CONVERSATIONS management
    class AUTH_LOGIN,AUTH_REFRESH,AUTH_VERIFY auth
```

### Chat API Flow

```mermaid
flowchart TD
    START([Client Request]) --> VALIDATE{Validate Input}
    VALIDATE -->|Valid| AUTH{Check Auth}
    VALIDATE -->|Invalid| ERROR_400[400 Bad Request]

    AUTH -->|Authorized| LOAD_CONTEXT[Load Chat Context]
    AUTH -->|Unauthorized| ERROR_401[401 Unauthorized]

    LOAD_CONTEXT --> SELECT_WORKFLOW{Select Workflow}
    SELECT_WORKFLOW --> CLASSIFICATION[classification-agent]
    SELECT_WORKFLOW --> BIKE_INSIGHTS[bike-insights<br/>Template workflow only]
    SELECT_WORKFLOW --> KNOWLEDGE_BASE[knowledge-base-agent]
    SELECT_WORKFLOW --> SQL_AGENT[sql-manipulation-agent]

    CLASSIFICATION --> PROCESS_MESSAGE[Process Multi-Agent Workflow]
    BIKE_INSIGHTS --> PROCESS_MESSAGE
    KNOWLEDGE_BASE --> PROCESS_MESSAGE
    SQL_AGENT --> PROCESS_MESSAGE

    PROCESS_MESSAGE --> LLM_CALL[Call Azure OpenAI]
    LLM_CALL --> SUCCESS{Success?}

    SUCCESS -->|Yes| SAVE_RESPONSE[Save Response]
    SUCCESS -->|No| ERROR_500[500 Server Error]

    SAVE_RESPONSE --> FORMAT_RESPONSE[Format Response]
    FORMAT_RESPONSE --> RETURN_200[200 Success]

    ERROR_400 --> END([End])
    ERROR_401 --> END
    ERROR_500 --> END
    RETURN_200 --> END

    classDef start fill:#c8e6c9
    classDef process fill:#e1f5fe
    classDef decision fill:#fff9c4
    classDef workflow fill:#f3e5f5
    classDef success fill:#dcedc8
    classDef error fill:#ffcdd2

    class START start
    class LOAD_CONTEXT,PROCESS_MESSAGE,LLM_CALL,SAVE_RESPONSE,FORMAT_RESPONSE process
    class VALIDATE,AUTH,SELECT_WORKFLOW,SUCCESS decision
    class BIKE_INSIGHTS,CLASSIFICATION,KNOWLEDGE_BASE,SQL_AGENT workflow
    class RETURN_200 success
    class ERROR_400,ERROR_401,ERROR_500 error
```

### Workflow API Flow

```mermaid
sequenceDiagram
    participant Client
    participant API as Chat API
    participant ChatService as Chat Service
    participant MultiAgentService as Multi-Agent Service
    participant Agent1 as Agent 1
    participant Agent2 as Agent 2
    participant LLM as Azure OpenAI

    Client->>API: POST /api/v1/chat
    Note over Client,API: {"user_prompt": "Help with SQL queries",\n"conversation_flow": "sql-manipulation-agent"}

    API->>ChatService: Process chat request
    ChatService->>MultiAgentService: Load conversation flow

    MultiAgentService->>Agent1: Start SQL agent processing
    MultiAgentService->>Agent2: Start classification if needed

    par Agent 1 Processing
        Agent1->>LLM: Parse natural language query
        LLM-->>Agent1: SQL generation
    and Agent 2 Processing (if needed)
        Agent2->>LLM: Classify query complexity
        LLM-->>Agent2: Query type classification
    end

    Agent1-->>MultiAgentService: Report SQL results
    Agent2-->>MultiAgentService: Report classification

    MultiAgentService->>MultiAgentService: Format results
    MultiAgentService-->>ChatService: Final response
    ChatService-->>API: Formatted response
    API-->>Client: 200 Success with SQL solution
```

## Authentication & Security

### Authentication Flow

```mermaid
graph TB
    subgraph "Authentication Methods"
        JWT_AUTH[JWT Authentication\nBearer Token]
        NO_AUTH[Authentication Disabled\nAnonymous Access]
    end

    subgraph "Security Features"
        HTTPS_TLS[HTTPS/TLS\nTransport Encryption]
        CONFIG_AUTH[Configurable Authentication\nEnable/Disable via Config]
        TOKEN_REFRESH[Token Refresh\nAutomatic Token Renewal]
    end

    subgraph "JWT Flow"
        LOGIN[POST /api/v1/auth/login\nUsername/Password]
        ACCESS_TOKEN[Access Token\n15 min expiry]
        REFRESH_TOKEN[Refresh Token\n7 days expiry]
        VERIFY[GET /api/v1/auth/verify\nValidate Token]
        REFRESH[POST /api/v1/auth/refresh\nRenew Access Token]
    end

    subgraph "Validation Steps"
        HEADER_CHECK[Authorization Header Check\nBearer Token]
        JWT_VERIFY[JWT Token Verification]
        CONFIG_CHECK[Check Auth Config]
        ACCESS_GRANTED[Access Granted]
    end

    JWT_AUTH --> LOGIN
    LOGIN --> ACCESS_TOKEN
    LOGIN --> REFRESH_TOKEN
    NO_AUTH --> ACCESS_GRANTED
    CONFIG_CHECK --> JWT_AUTH
    CONFIG_CHECK --> NO_AUTH

    ACCESS_TOKEN --> HEADER_CHECK
    HEADER_CHECK --> JWT_VERIFY
    JWT_VERIFY --> ACCESS_GRANTED
    JWT_VERIFY --> VERIFY

    REFRESH_TOKEN --> REFRESH
    REFRESH --> ACCESS_TOKEN

    HTTPS_TLS --> HEADER_CHECK
    CONFIG_AUTH --> CONFIG_CHECK
    TOKEN_REFRESH --> REFRESH

    classDef auth fill:#e8f5e8
    classDef security fill:#fff3e0
    classDef validation fill:#e3f2fd
    classDef jwt fill:#ffe0b2

    class JWT_AUTH,NO_AUTH auth
    class HTTPS_TLS,CONFIG_AUTH,TOKEN_REFRESH security
    class HEADER_CHECK,JWT_VERIFY,CONFIG_CHECK,ACCESS_GRANTED validation
    class LOGIN,ACCESS_TOKEN,REFRESH_TOKEN,VERIFY,REFRESH jwt
```

### Getting Started with the API

The Insight Ingenious API provides powerful endpoints for creating and managing AI-powered conversation workflows programmatically.

### Base API Information
- **Base URL**: `http://localhost:8000` (when using `ingen serve --port 8000`)
- **Content-Type**: `application/json`
- **Authentication**: JWT Bearer Token (configurable - disabled by default)

### [Workflow API](./WORKFLOWS.md)
Complete documentation for all available workflow endpoints, including:
- Classification and routing workflows
- Multi-agent bike sales analysis (template workflow)
- Knowledge base search and retrieval
- SQL query generation and execution

### Authentication API Endpoints

#### JWT Login
```bash
POST /api/v1/auth/login
```
Authenticate a user and receive JWT tokens.

**Request Body:**
```json
{
  "username": "user@example.com",
  "password": "password123"
}
```

**Response:**
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "bearer"
}
```

#### Refresh Token
```bash
POST /api/v1/auth/refresh
```
Refresh an expired access token using a valid refresh token.

**Request Body:**
```json
{
  "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

**Response:**
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "bearer"
}
```

#### Verify Token
```bash
GET /api/v1/auth/verify
```
Verify if a JWT token is valid. The token must be provided in the Authorization header using Bearer scheme.

**Headers:**
```
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

**Response:**
```json
{
  "username": "user@example.com",
  "valid": true
}
```

### Core API Endpoints

#### Health Check
```bash
GET /api/v1/health
```
Returns the health status of the API service.

#### System Diagnostic
```bash
GET /api/v1/diagnostic
```
Returns diagnostic information about the system including directory paths for prompts, data, output, and events.

#### List Available Workflows
```bash
GET /api/v1/workflows
```
Returns a list of all available workflow types and their configurations.

#### Get Conversation History
```bash
GET /api/v1/conversations/{thread_id}
```
Retrieves all messages from a specific conversation thread.

**Parameters:**
- `thread_id` (path): The unique identifier of the conversation thread

**Response:**
```json
[
  {
    "id": "msg-123",
    "thread_id": "thread-456",
    "user_id": "user-789",
    "role": "user|assistant|system",
    "content": "User message or agent response",
    "name": "Agent name (optional)",
    "positive_feedback": null,
    "content_filter_results": null,
    "tool_calls": [],
    "created_at": "2025-07-04T12:00:00Z",
    "updated_at": "2025-07-04T12:00:00Z"
  }
]
```

#### Submit Message Feedback
```bash
PUT /api/v1/messages/{message_id}/feedback
```
Submit feedback for a specific message.

**Parameters:**
- `message_id` (path): The unique identifier of the message

**Request Body:**
```json
{
  "thread_id": "thread-456",
  "message_id": "msg-123",
  "user_id": "user-789 (optional)",
  "positive_feedback": true
}
```

**Response:**
```json
{
  "message": "Feedback submitted successfully"
}
```

### Workflow Naming Conventions

Insight Ingenious supports both hyphenated and underscored workflow names for backward compatibility:
- `classification-agent` or `classification_agent`
- `bike-insights` or `bike_insights` (template only)
- `knowledge-base-agent` or `knowledge_base_agent`
- `sql-manipulation-agent` or `sql_manipulation_agent`

**Note**: `bike-insights` is only available in projects created with `uv run ingen init`, not in the core library.

### Common API Patterns

#### Making API Requests
All API requests should include appropriate headers:

```bash
curl -X POST http://localhost:8000/api/v1/chat \
  -H "Content-Type: application/json" \
  -d '{"user_prompt": "Hello", "conversation_flow": "classification-agent"}'
```

With JWT authentication enabled:
```bash
# First login to get tokens
curl -X POST http://localhost:8000/api/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username": "user@example.com", "password": "password123"}'

# Use the access token for requests
curl -X POST http://localhost:8000/api/v1/chat \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." \
  -d '{"user_prompt": "Hello", "conversation_flow": "classification-agent"}'
```

#### Response Format
API responses are returned directly without a wrapper. For chat endpoints, the response includes:

```json
{
  "thread_id": "thread-456",
  "message_id": "msg-123",
  "agent_response": "The AI agent's response...",
  "followup_questions": {},
  "token_count": 245,
  "max_token_count": 1024,
  "topic": "General",
  "memory_summary": "Conversation summary",
  "event_type": "chat"
}
```

Error responses follow FastAPI's standard format:
```json
{
  "detail": "Error message describing what went wrong"
}
```

## Integration Examples

### Python Integration
```python
import requests

class IngeniousAPIClient:
    def __init__(self, base_url="http://localhost:8000"):
        self.base_url = base_url
        self.access_token = None

    def login(self, username, password):
        """Login and store access token"""
        response = requests.post(
            f"{self.base_url}/api/v1/auth/login",
            json={"username": username, "password": password}
        )
        if response.status_code == 200:
            data = response.json()
            self.access_token = data["access_token"]
            return True
        return False

    def call_chat_api(self, user_prompt, conversation_flow, thread_id=None):
        """Call the chat API with authentication"""
        headers = {"Content-Type": "application/json"}
        if self.access_token:
            headers["Authorization"] = f"Bearer {self.access_token}"

        payload = {
            "user_prompt": user_prompt,
            "conversation_flow": conversation_flow
        }
        if thread_id:
            payload["thread_id"] = thread_id

        response = requests.post(
            f"{self.base_url}/api/v1/chat",
            json=payload,
            headers=headers
        )
        return response.json()

# Example usage
client = IngeniousAPIClient()
if client.login("user@example.com", "password123"):
    result = client.call_chat_api("Hello", "classification-agent")
```

### JavaScript Integration
```javascript
class IngeniousAPIClient {
    constructor(baseUrl = 'http://localhost:8000') {
        this.baseUrl = baseUrl;
        this.accessToken = null;
    }

    async login(username, password) {
        const response = await fetch(`${this.baseUrl}/api/v1/auth/login`, {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({ username, password })
        });

        if (response.ok) {
            const data = await response.json();
            this.accessToken = data.access_token;
            return true;
        }
        return false;
    }

    async callChatAPI(userPrompt, conversationFlow, threadId = null) {
        const headers = { 'Content-Type': 'application/json' };
        if (this.accessToken) {
            headers['Authorization'] = `Bearer ${this.accessToken}`;
        }

        const payload = { user_prompt: userPrompt, conversation_flow: conversationFlow };
        if (threadId) {
            payload.thread_id = threadId;
        }

        const response = await fetch(`${this.baseUrl}/api/v1/chat`, {
            method: 'POST',
            headers,
            body: JSON.stringify(payload)
        });

        return await response.json();
    }
}

// Example usage
const client = new IngeniousAPIClient();
if (await client.login('user@example.com', 'password123')) {
    const result = await client.callChatAPI('Hello', 'classification-agent');
}
```

## Prompts API

The Prompts API provides endpoints for managing prompt templates used by conversation workflows. These templates are stored in the configured file storage backend (local or Azure Blob Storage).

### Key Features

- **Template Management**: Create, read, update, and delete prompt templates
- **Version Control**: Organize prompts by revision IDs for version management
- **Cloud Storage**: Seamlessly works with both local and Azure Blob Storage backends
- **Real-time Updates**: Changes take effect immediately in active workflows

### Base Endpoints

#### List All Revisions
```bash
GET /api/v1/revisions/list
```

Lists all available prompt revision IDs.

**Response:**
```json
{
  "revisions": ["v1.0", "v1.1", "v2.0", "development"],
  "count": 4
}
```

#### List Workflows with Prompts
```bash
GET /api/v1/workflows/list
```

Lists all workflows that have prompt templates.

**Response:**
```json
{
  "workflows": [
    "classification-agent",
    "knowledge-base-agent",
    "sql-manipulation-agent",
    "bike-insights"
  ],
  "count": 4
}
```

#### List Prompt Files
```bash
GET /api/v1/prompts/list/{revision_id}
```

Lists all prompt template files for a specific revision.

**Parameters:**
- `revision_id` (path): The revision identifier (e.g., "v1.0", "main", "development")

**Response:**
```json
{
  "revision_id": "v1.0",
  "actual_revision_used": "v1.0",
  "normalized_revision_id": "v1.0",
  "files": [
    "user_prompt.md",
    "system_prompt.md",
    "classification_prompt.jinja",
    "sql_generation_prompt.md"
  ],
  "count": 4,
  "attempted_revisions": ["v1.0"]
}
```

**Example:**
```bash
curl "http://localhost:8000/api/v1/prompts/list/v1.0"
```

#### View Prompt Content
```bash
GET /api/v1/prompts/view/{revision_id}/{filename}
```

Retrieves the content of a specific prompt template file.

**Parameters:**
- `revision_id` (path): The revision identifier
- `filename` (path): The name of the prompt file

**Response:** Raw text content of the prompt file

**Example:**
```bash
curl "http://localhost:8000/api/v1/prompts/view/v1.0/user_prompt.md"
```

#### Update Prompt Content
```bash
POST /api/v1/prompts/update/{revision_id}/{filename}
```

Updates or creates a prompt template file with new content.

**Parameters:**
- `revision_id` (path): The revision identifier
- `filename` (path): The name of the prompt file

**Request Body:**
```json
{
  "content": "# Updated Prompt Template\n\nYour prompt content here..."
}
```

**Response:**
```json
{
  "message": "File updated successfully"
}
```

**Example:**
```bash
curl -X POST "http://localhost:8000/api/v1/prompts/update/v1.0/user_prompt.md" \
  -H "Content-Type: application/json" \
  -d '{"content": "# My Updated Prompt\n\nContext: {{ context }}\nQuestion: {{ question }}"}'
```

### Integration with Storage Backends

The Prompts API automatically uses the configured file storage backend:

#### Local Storage
- Files stored in `.files/prompts/{revision_id}/` directory
- Direct filesystem access for development and testing

#### Azure Blob Storage
- Files stored in Azure Blob Storage containers
- Scalable, distributed storage for production environments
- Automatic authentication using configured credentials

### Workflow Integration

Prompt templates are automatically loaded by conversation workflows:

```python
# Workflows automatically use updated prompts
from ingenious.utils.conversation_builder import Sync_Prompt_Templates

# Sync templates from storage (local or cloud)
await Sync_Prompt_Templates(config, revision_id="v1.0")
```

### Template Format Support

The API supports various template formats:

- **Markdown (.md)**: For human-readable prompts
- **Jinja2 (.jinja)**: For templated prompts with variables
- **Text (.txt)**: For simple text prompts

### Example Workflow

1. **Create a new prompt revision:**
   ```bash
   # Create user prompt
   curl -X POST "http://localhost:8000/api/v1/prompts/update/v2.0/user_prompt.md" \
     -H "Content-Type: application/json" \
     -d '{"content": "# Enhanced User Prompt\n\nContext: {{ context }}\nUser Input: {{ user_question }}\n\nProvide a comprehensive response."}'
   ```

2. **List available prompts:**
   ```bash
   curl "http://localhost:8000/api/v1/prompts/list/v2.0"
   ```

3. **View the prompt content:**
   ```bash
   curl "http://localhost:8000/api/v1/prompts/view/v2.0/user_prompt.md"
   ```

4. **Update configuration to use new revision:**
   ```yaml
   # In your workflow configuration
   prompt_revision: "v2.0"
   ```

### Python Integration Example

```python
import requests

class PromptsAPIClient:
    def __init__(self, base_url="http://localhost:8000"):
        self.base_url = base_url

    def list_prompts(self, revision_id):
        """List all prompt files for a revision"""
        response = requests.get(f"{self.base_url}/api/v1/prompts/list/{revision_id}")
        data = response.json()
        # Return just the files list for backward compatibility
        return data.get("files", [])

    def get_prompt(self, revision_id, filename):
        """Get content of a specific prompt file"""
        response = requests.get(f"{self.base_url}/api/v1/prompts/view/{revision_id}/{filename}")
        return response.text

    def update_prompt(self, revision_id, filename, content):
        """Update or create a prompt file"""
        response = requests.post(
            f"{self.base_url}/api/v1/prompts/update/{revision_id}/{filename}",
            json={"content": content}
        )
        return response.json()

# Usage
client = PromptsAPIClient()

# List prompts
prompts = client.list_prompts("v1.0")
print(f"Available prompts: {prompts}")

# Get prompt content
content = client.get_prompt("v1.0", "user_prompt.md")
print(f"Prompt content: {content}")

# Update prompt
result = client.update_prompt("v1.0", "user_prompt.md", "# New Prompt Content")
print(f"Update result: {result}")
```

### Best Practices

1. **Use semantic versioning** for revision IDs (e.g., "v1.0", "v1.1", "v2.0")
2. **Test prompts** in development revisions before promoting to production
3. **Include templates variables** for dynamic content (e.g., `{{ context }}`, `{{ user_question }}`)
4. **Document prompt changes** in commit messages or change logs
5. **Backup important prompts** before making major changes

### Error Handling

The Prompts API returns standard HTTP status codes:

- `200 OK` - Successful request
- `404 Not Found` - Prompt file or revision not found
- `400 Bad Request` - Invalid request body or parameters
- `500 Internal Server Error` - Storage backend error

Example error response:
```json
{
  "detail": "Prompt file not found: user_prompt.md in revision v1.0"
}
```

## Error Handling

The API uses standard HTTP status codes and provides detailed error messages:

- `200 OK` - Successful request
- `400 Bad Request` - Invalid request parameters
- `401 Unauthorized` - Missing or invalid authentication
- `404 Not Found` - Endpoint or resource not found
- `406 Not Acceptable` - Content filter error
- `413 Request Entity Too Large` - Token limit exceeded
- `500 Internal Server Error` - Server-side error
- `503 Service Unavailable` - System degraded

Example error response:
```json
{
  "detail": "Invalid workflow type specified: unknown-workflow"
}
```

## Additional Resources

- [Workflow API Documentation](./WORKFLOWS.md)
- [Configuration Guide](../getting-started/configuration.md)
- [Development Setup](../development/README.md)
- [CLI Reference](../CLI_REFERENCE.md)

## Need Help?

- Check the [troubleshooting guide](../troubleshooting/README.md)
- Review the [workflow examples](./WORKFLOWS.md)
- Open an issue on [GitHub](https://github.com/Insight-Services-APAC/ingenious/issues)
