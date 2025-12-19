# AutoMart-AI-Agents

```text
┌─────────────────────────────────────────────────────────────────────────────┐
│                              automart-mcp-server                            │
│                         (REST API + MCP Tools)                              │
└─────────────────────────────────────────┬───────────────────────────────────┘
                                          │
                                          ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                          automart-workflow-engine                           │
│                                                                             │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐                     │
│  │  Sequential │    │  Parallel   │    │  Consensus  │  (e.g., majority    │
│  │  Workflow   │    │  Workflow   │    │  Workflow   │   vote from LLMs)   │
│  └─────────────┘    └─────────────┘    └─────────────┘                     │
└─────────────────────────────────────────┬───────────────────────────────────┘
                                          │
                                          ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                            automart-ai-ml-core                              │
│                                                                             │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │                         AIProvider Interface                          │  │
│  │  - chat(prompt) → Response                                            │  │
│  │  - chatWithTools(prompt, tools) → Response                            │  │
│  │  - embed(text) → float[]                                              │  │
│  │  - generateImage(prompt) → Image                                      │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                                                             │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │                      WorkflowEngine Interface                         │  │
│  │  - executeSequential(tasks) → Results                                 │  │
│  │  - executeParallel(tasks) → Results                                   │  │
│  │  - executeWithStrategy(tasks, strategy) → Results                     │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                                                             │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │                         Common Models                                 │  │
│  │  - AIRequest, AIResponse, ChatMessage, ToolDefinition, etc.           │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────────┘
                                          │
                    ┌─────────────────────┼─────────────────────┐
                    │                     │                     │
                    ▼                     ▼                     ▼
┌─────────────────────────┐ ┌─────────────────────────┐ ┌─────────────────────────┐
│ automart-ai-ml-openai   │ │ automart-ai-ml-vertex   │ │ automart-ai-ml-bedrock  │
│                         │ │                         │ │                         │
│ - OpenAIProvider        │ │ - VertexAIProvider      │ │ - BedrockProvider       │
│ - Uses OpenAI Java SDK  │ │ - Uses Google Cloud SDK │ │ - Uses AWS SDK          │
└─────────────────────────┘ └─────────────────────────┘ └─────────────────────────┘
                                          │
                                          ▼
                            ┌─────────────────────────┐
                            │ automart-ai-ml-azure    │
                            │                         │
                            │ - AzureOpenAIProvider   │
                            │ - Uses Azure SDK        │
                            └─────────────────────────┘
```