---
title: "Workflow Requirements"
layout: single
permalink: /workflows/
sidebar:
  nav: "docs"
toc: true
toc_label: "Workflow Types"
toc_icon: "project-diagram"
---

This guide outlines the configuration requirements for each conversation workflow in Insight Ingenious - an enterprise-grade Python library for quickly setting up APIs to interact with AI Agents. Understanding these requirements will help you determine what Azure services and configurations are needed for each workflow, along with available debugging and customization options.

## Workflow Architecture Overview

```mermaid
graph TB
    subgraph "Core Library Workflows"
        CLASSIFICATION[Classification Agent<br/>Route to specialists]
        KNOWLEDGE[Knowledge Base Agent<br/>Information retrieval]
        SQL[SQL Manipulation Agent<br/>Database queries]
    end

    subgraph "Template Workflows"
        BIKE_INSIGHTS[Bike Insights<br/>Multi-agent analysis]
    end

    subgraph "Configuration Levels"
        MINIMAL[Minimal Config<br/>Azure OpenAI only]
        LOCAL_IMPL[Local Implementations<br/>ChromaDB + SQLite]
        AZURE_EXPERIMENTAL[Azure Services<br/>Experimental/Unstable]
    end

    subgraph "External Dependencies"
        AZURE_OPENAI[Azure OpenAI]
        CHROMADB[ChromaDB<br/>Local Vector DB]
        SQLITE[SQLite<br/>Local Database]
        AZURE_SEARCH[Azure Cognitive Search<br/>Production-ready]
        AZURE_SQL[Azure SQL Database<br/>Production-ready]
    end

    CLASSIFICATION --> MINIMAL
    BIKE_INSIGHTS --> MINIMAL

    KNOWLEDGE --> LOCAL_IMPL
    SQL --> LOCAL_IMPL

    MINIMAL --> AZURE_OPENAI
    LOCAL_IMPL --> AZURE_OPENAI
    LOCAL_IMPL --> CHROMADB
    LOCAL_IMPL --> SQLITE

    AZURE_EXPERIMENTAL --> AZURE_SEARCH
    AZURE_EXPERIMENTAL --> AZURE_SQL

    classDef workflow fill:#e3f2fd
    classDef config fill:#f1f8e9
    classDef external fill:#fff3e0

    class CLASSIFICATION,KNOWLEDGE,SQL,BIKE_INSIGHTS workflow
    class MINIMAL,LOCAL_IMPL,AZURE_EXPERIMENTAL config
    class AZURE_OPENAI,CHROMADB,SQLITE,AZURE_SEARCH,AZURE_SQL external
```

## Core vs Template Workflows

### Core Library Workflows
These workflows are built into the Insight Ingenious core library and are always available:
- **classification-agent** - Routes user queries to appropriate specialized agents
- **knowledge-base-agent** - Searches and retrieves information from knowledge bases
- **sql-manipulation-agent** - Executes SQL queries based on natural language

### Template Workflows
These workflows are provided as examples in the `ingenious_extensions_template` when you run `uv run ingen init`:
- **bike-insights** - Multi-agent bike sales analysis (the "Hello World" example)

**Important**: Template workflows like `bike-insights` are only available in projects created with `uv run ingen init`, not in the core library.

## Implementation Stability Guide

###  Production-Ready Implementations
- **Local ChromaDB** (knowledge-base-agent): Vector database for knowledge search
- **Local SQLite** (sql-manipulation-agent): Database for SQL queries
- **Azure OpenAI**: Required across all workflows
- **Azure Cognitive Search** (knowledge-base-agent): Cloud-based knowledge search
- **Azure SQL Database** (sql-manipulation-agent): Cloud-based database

**Note**: Both local (ChromaDB + SQLite) and Azure (Azure Search + Azure SQL) implementations are production-ready. Choose based on your infrastructure requirements.

## Detailed Workflow Flows

###  Classification Agent Workflow

```mermaid
sequenceDiagram
    participant User
    participant API
    participant Coordinator
    participant ClassificationAgent as Classification Agent
    participant GeneralAgent as General Classification
    participant KnowledgeAgent as Knowledge Base Agent
    participant SQLAgent as SQL Agent
    participant AzureOpenAI as Azure OpenAI

    User->>API: "Help me with database queries"
    API->>Coordinator: Initialize classification-agent workflow

    Coordinator->>ClassificationAgent: Classify user intent
    ClassificationAgent->>AzureOpenAI: Analyze query type
    AzureOpenAI-->>ClassificationAgent: Intent: SQL Query

    ClassificationAgent->>Coordinator: Route to SQL Agent
    Coordinator->>SQLAgent: Handle database query
    SQLAgent->>AzureOpenAI: Generate SQL solution
    AzureOpenAI-->>SQLAgent: SQL query & explanation

    SQLAgent-->>Coordinator: Formatted response
    Coordinator-->>API: Complete analysis
    API-->>User: SQL solution with explanation
```

### Classification Agent Workflow Flow

```mermaid
flowchart TD
    START([User Input]) --> CLASSIFY{Classify Intent}

    CLASSIFY -->|Product Inquiry| PRODUCT_FLOW[Product Query Flow]
    CLASSIFY -->|Purchase Question| PURCHASE_FLOW[Purchase Query Flow]
    CLASSIFY -->|Support Issue| SUPPORT_FLOW[Support Query Flow]
    CLASSIFY -->|Undefined| UNDEFINED_FLOW[Undefined Query Flow]

    PRODUCT_FLOW --> PRODUCT_AGENT[Product Agent]
    PURCHASE_FLOW --> PURCHASE_AGENT[Purchase Agent]
    SUPPORT_FLOW --> SUPPORT_AGENT[Support Agent]
    UNDEFINED_FLOW --> UNDEFINED_AGENT[Undefined Agent]

    PRODUCT_AGENT --> RESPONSE[Formatted Response]
    PURCHASE_AGENT --> RESPONSE
    SUPPORT_AGENT --> RESPONSE
    UNDEFINED_AGENT --> RESPONSE
    KNOWLEDGE_AGENT --> RESPONSE
    SQL_AGENT --> RESPONSE
    CLASSIFICATION_AGENT --> RESPONSE

    RESPONSE --> FINISH([End])

    classDef start fill:#c8e6c9
    classDef decision fill:#fff9c4
    classDef workflow fill:#e1f5fe
    classDef agent fill:#f3e5f5
    classDef finish fill:#ffcdd2

    class START start
    class CLASSIFY decision
    class PRODUCT_FLOW,PURCHASE_FLOW,SUPPORT_FLOW,UNDEFINED_FLOW workflow
    class PRODUCT_AGENT,PURCHASE_AGENT,SUPPORT_AGENT,UNDEFINED_AGENT agent
    class RESPONSE,FINISH finish
```

### Knowledge Base Workflow

```mermaid
graph TB
    subgraph "Input Processing"
        USER_QUERY[User Query]
        INTENT_ANALYSIS[Intent Analysis]
        QUERY_ENHANCEMENT[Query Enhancement]
    end

    subgraph "Search & Retrieval"
        CHROMADB["ChromaDB (Local)"]
        VECTOR_SEARCH[Vector Search]
        KEYWORD_SEARCH[Keyword Search]
        HYBRID_SEARCH[Hybrid Search]
    end

    subgraph "Content Processing"
        RELEVANCE_SCORING[Relevance Scoring]
        CONTENT_RANKING[Content Ranking]
        CONTEXT_EXTRACTION[Context Extraction]
    end

    subgraph "AI Processing"
        AZURE_OPENAI[Azure OpenAI]
        CONTEXT_SYNTHESIS[Context Synthesis]
        RESPONSE_GENERATION[Response Generation]
    end

    USER_QUERY --> INTENT_ANALYSIS
    INTENT_ANALYSIS --> QUERY_ENHANCEMENT
    QUERY_ENHANCEMENT --> CHROMADB

    CHROMADB --> VECTOR_SEARCH
    CHROMADB --> KEYWORD_SEARCH
    CHROMADB --> HYBRID_SEARCH

    VECTOR_SEARCH --> RELEVANCE_SCORING
    KEYWORD_SEARCH --> RELEVANCE_SCORING
    HYBRID_SEARCH --> RELEVANCE_SCORING

    RELEVANCE_SCORING --> CONTENT_RANKING
    CONTENT_RANKING --> CONTEXT_EXTRACTION
    CONTEXT_EXTRACTION --> AZURE_OPENAI

    AZURE_OPENAI --> CONTEXT_SYNTHESIS
    CONTEXT_SYNTHESIS --> RESPONSE_GENERATION

    classDef input fill:#e8f5e8
    classDef search fill:#fff3e0
    classDef processing fill:#e3f2fd
    classDef ai fill:#fce4ec

    class USER_QUERY,INTENT_ANALYSIS,QUERY_ENHANCEMENT input
    class CHROMADB,VECTOR_SEARCH,KEYWORD_SEARCH,HYBRID_SEARCH search
    class RELEVANCE_SCORING,CONTENT_RANKING,CONTEXT_EXTRACTION processing
    class AZURE_OPENAI,CONTEXT_SYNTHESIS,RESPONSE_GENERATION ai
```

### SQL Manipulation Workflow

```mermaid
sequenceDiagram
    participant User
    participant SQLAgent
    participant AzureOpenAI
    participant Database

    User->>SQLAgent: "Show me sales by region"
    SQLAgent->>AzureOpenAI: Convert natural language to SQL
    AzureOpenAI-->>SQLAgent: Generated SQL query

    SQLAgent->>Database: Execute SQL query
    Note over Database: Supports both:<br/>- Azure SQL Database<br/>- Local SQLite
    Database-->>SQLAgent: Query results

    SQLAgent->>AzureOpenAI: Format results for user
    AzureOpenAI-->>SQLAgent: Natural language response

    SQLAgent-->>User: "Sales by region analysis"

    Note over SQLAgent,Database: Configuration determines:<br/>- Azure SQL vs SQLite<br/>- Database connection details<br/>- Query timeout settings
```

## Configuration Requirements by Workflow

### Core Library Workflows (Azure OpenAI only)

These workflows are included in the core library and only require basic Azure OpenAI configuration:

#### Classification Agent
Routes input to specialized agents based on content analysis.

```mermaid
graph LR
    subgraph "Required Services"
        AZURE_OPENAI[Azure OpenAI<br/>Intent Classification]
    end

    subgraph "Configuration Method"
        ENV_VARS[Environment Variables<br/>INGENIOUS_ prefixed]
        ENV_FILE[.env file<br/>Local development]
    end

    ENV_VARS --> AZURE_OPENAI
    ENV_FILE --> AZURE_OPENAI

    classDef service fill:#e3f2fd
    classDef config fill:#f1f8e9

    class AZURE_OPENAI service
    class ENV_VARS,ENV_FILE config
```

**Required Configuration:**
```bash
# Environment variables for classification agent
INGENIOUS_PROFILE=dev
INGENIOUS_MODELS__0__MODEL=gpt-4o-mini
INGENIOUS_MODELS__0__API_TYPE=rest
INGENIOUS_MODELS__0__API_VERSION=2024-12-01-preview
INGENIOUS_MODELS__0__DEPLOYMENT=gpt-4o-mini
INGENIOUS_MODELS__0__API_KEY=your-api-key
INGENIOUS_MODELS__0__BASE_URL=https://your-resource.openai.azure.com/
INGENIOUS_CHAT_SERVICE__TYPE=multi_agent
```

### Template-Based Workflows (Azure OpenAI only)

#### Bike Insights ("Hello World" Template)
Sample domain-specific workflow for bike sales analysis. Available in the `ingenious_extensions_template` when you run `ingen init`.

> **Note:** This workflow exists as a template example in `ingenious_extensions_template/`, not as a core workflow. It demonstrates how to build custom domain-specific workflows and is the recommended "Hello World" example for new users.

```mermaid
graph TB
    subgraph "Required Services"
        AZURE_OPENAI[Azure OpenAI\nMulti-Agent Processing]
    end

    subgraph "Template Files"
        BIKE_DATA[Bike Sales Data\nJSON Sample Files]
        BIKE_MODELS[Bike Data Models\nPydantic Schemas]
    end

    subgraph "Template Agents"
        BIKE_AGENT[Bike Analysis Agent\nTemplate Example]
        AGENT_FLOW[Conversation Flow\nTemplate Pattern]
    end

    AZURE_OPENAI --> BIKE_AGENT
    BIKE_DATA --> BIKE_AGENT
    BIKE_MODELS --> BIKE_AGENT
    BIKE_AGENT --> AGENT_FLOW

    classDef service fill:#e3f2fd
    classDef template fill:#f1f8e9
    classDef agent fill:#fff3e0

    class AZURE_OPENAI service
    class BIKE_DATA,BIKE_MODELS template
    class BIKE_AGENT,AGENT_FLOW agent
```

### Core Library Workflows (Local Implementation - Stable)

#### Knowledge Base Agent
Search and retrieve information from knowledge bases using local ChromaDB (stable) or Azure Search (experimental).

> **Note**: Both local ChromaDB and Azure Search implementations are production-ready. ChromaDB requires no additional configuration and is ideal for development.

```mermaid
graph TB
    subgraph "Required Services"
        AZURE_OPENAI[ Azure OpenAI<br/>Response Generation]
        AZURE_SEARCH[Azure Cognitive Search<br/>Document Retrieval]
    end

    subgraph "Knowledge Sources"
        DOCUMENTS[Documents<br/>PDFs, Word, Text]
        WEBSITES[Web Content<br/>Scraped Pages]
        DATABASES[Structured Data<br/>FAQ, Knowledge Base]
    end

    subgraph "Search Capabilities"
        VECTOR_SEARCH[Vector Search<br/>Semantic Similarity]
        KEYWORD_SEARCH[Keyword Search<br/>Full-Text Search]
        HYBRID_SEARCH[Hybrid Search<br/>Combined Approach]
    end

    DOCUMENTS --> AZURE_SEARCH
    WEBSITES --> AZURE_SEARCH
    DATABASES --> AZURE_SEARCH

    AZURE_SEARCH --> VECTOR_SEARCH
    AZURE_SEARCH --> KEYWORD_SEARCH
    AZURE_SEARCH --> HYBRID_SEARCH

    VECTOR_SEARCH --> AZURE_OPENAI
    KEYWORD_SEARCH --> AZURE_OPENAI
    HYBRID_SEARCH --> AZURE_OPENAI

    classDef service fill:#e3f2fd
    classDef source fill:#f1f8e9
    classDef search fill:#fff3e0

    class AZURE_OPENAI,AZURE_SEARCH service
    class DOCUMENTS,WEBSITES,DATABASES source
    class VECTOR_SEARCH,KEYWORD_SEARCH,HYBRID_SEARCH search
```

**Configuration Options:**

**Recommended: Local ChromaDB (Stable - No additional configuration needed)**
```bash
# No additional configuration required!
# Uses local ChromaDB for vector storage
# Documents stored in ./.tmp/knowledge_base/

# To use knowledge-base-agent, ensure these dependencies are installed:
uv add chromadb aiofiles autogen-ext
```

**Alternative: Azure Search (Production-ready)**
```bash
# Additional Azure Search configuration
INGENIOUS_AZURE_SEARCH_SERVICES__0__ENDPOINT=https://your-search-service.search.windows.net
INGENIOUS_AZURE_SEARCH_SERVICES__0__KEY=your-search-api-key
```

> **Note**: Both implementations are production-ready. ChromaDB is ideal for development and smaller deployments, while Azure Search provides enterprise-scale capabilities.

### Core Library Workflows (Database Required)

#### SQL Manipulation Agent
Execute SQL queries on Azure SQL or local databases.

```mermaid
graph TB
    subgraph "Required Services"
        AZURE_OPENAI[ Azure OpenAI<br/>Query Generation & Formatting]
        DATABASE[Database<br/>SQL Server, PostgreSQL, MySQL]
    end

    subgraph "Security Layer"
        QUERY_VALIDATOR[Query Validator<br/>SQL Injection Prevention]
        PERMISSION_CHECK[Permission Check<br/>Table Access Control]
        OPERATION_FILTER[Operation Filter<br/>Read-Only Enforcement]
    end

    subgraph "Query Processing"
        NL_TO_SQL[Natural Language to SQL]
        RESULT_FORMATTER[Result Formatter]
        ERROR_HANDLER[Error Handler]
    end

    AZURE_OPENAI --> NL_TO_SQL
    NL_TO_SQL --> QUERY_VALIDATOR
    QUERY_VALIDATOR --> PERMISSION_CHECK
    PERMISSION_CHECK --> OPERATION_FILTER
    OPERATION_FILTER --> DATABASE
    DATABASE --> RESULT_FORMATTER
    RESULT_FORMATTER --> AZURE_OPENAI

    classDef service fill:#e3f2fd
    classDef security fill:#ffcdd2
    classDef processing fill:#f1f8e9

    class AZURE_OPENAI,DATABASE service
    class QUERY_VALIDATOR,PERMISSION_CHECK,OPERATION_FILTER security
    class NL_TO_SQL,RESULT_FORMATTER,ERROR_HANDLER processing
```

**Additional Configuration Required:**

**Local SQLite (Recommended - Stable):**
```bash
# Local SQLite configuration
INGENIOUS_LOCAL_SQL_DB__DATABASE_PATH=/tmp/sample_sql.db

# No additional dependencies required - SQLite support is built-in
```

**Azure SQL (Production-ready):**
```bash
# Azure SQL configuration
INGENIOUS_AZURE_SQL_SERVICES__DATABASE_NAME=your-database-name
INGENIOUS_AZURE_SQL_SERVICES__TABLE_NAME=your-table-name
INGENIOUS_AZURE_SQL_SERVICES__DATABASE_CONNECTION_STRING="Driver={ODBC Driver 18 for SQL Server};Server=tcp:your-server.database.windows.net,1433;Database=your-database;Uid=your-username;Pwd=your-password;Encrypt=yes;TrustServerCertificate=no;Connection Timeout=30;"
```

> **Note**: Both SQLite and Azure SQL implementations are production-ready. SQLite is ideal for development and testing.

## Workflow Selection Guide

### Choosing the Right Workflow

```mermaid
flowchart TD
    START(["What do you want to do?"]) --> DECISION{"Choose your use case"}

    DECISION -->|Route user queries to different specialists| CLASSIFICATION["Classification Agent"]
    DECISION -->|Analyze business data with multiple perspectives| BIKE_INSIGHTS["Bike Insights (Template only)"]
    DECISION -->|Search through documents and knowledge| KNOWLEDGE["Knowledge Base Agent"]
    DECISION -->|Query databases with natural language| SQL["SQL Manipulation"]

    CLASSIFICATION --> SETUP_MINIMAL["Minimal Setup Azure OpenAI only"]
    BIKE_INSIGHTS --> SETUP_MINIMAL

    KNOWLEDGE --> SETUP_SEARCH["Search Setup + Azure Cognitive Search"]

    SQL --> SETUP_DATABASE["Database Setup + Database Connection"]

    SETUP_MINIMAL --> READY["Ready to Use"]
    SETUP_SEARCH --> READY
    SETUP_DATABASE --> READY

    classDef start fill:#c8e6c9
    classDef decision fill:#fff9c4
    classDef workflow fill:#e1f5fe
    classDef setup fill:#f3e5f5
    classDef ready fill:#dcedc8

    class START start
    class DECISION decision
    class CLASSIFICATION,BIKE_INSIGHTS,KNOWLEDGE,SQL workflow
    class SETUP_MINIMAL,SETUP_SEARCH,SETUP_DATABASE setup
    class READY ready
```

## Next Steps

1. **Choose Your Workflow**: Select the workflow that best fits your use case
2. **Configure Services**: Set up the required Azure services and configuration
3. **Test Setup**: Validate your configuration with sample queries
4. **Deploy**: Launch your workflow in your preferred environment
5. **Monitor**: Track performance and optimize as needed

For detailed setup instructions, see:
- [Configuration Guide](../getting-started/configuration.md) - Complete setup instructions
- [Getting Started](../getting-started/README.md) - Quick start tutorial
- [Development Guide](../development/README.md) - Advanced customization
- [API Documentation](../api/README.md) - Integration details
