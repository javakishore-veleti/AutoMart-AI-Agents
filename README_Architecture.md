# AutoMart AI Agents - Technical Architecture

## Executive Summary

AutoMart AI Agents is built on a modular, cloud-agnostic architecture that separates AI provider implementations from business logic. The system supports multiple AI providers (OpenAI, Google Vertex AI, AWS Bedrock, Azure OpenAI) through a unified abstraction layer, orchestrated by a flexible workflow engine capable of sequential, parallel, and consensus-based execution strategies.

## Architectural Principles

### Provider Agnosticism
The platform treats AI providers as interchangeable implementations of common interfaces. Business logic never depends on a specific provider, enabling:
- Easy provider switching based on cost, performance, or availability
- Multi-provider strategies for improved reliability
- Gradual migration between providers without code changes
- A/B testing across different AI models

### Separation of Concerns
Clear boundaries exist between:
- **Interface definitions** (what capabilities exist)
- **Provider implementations** (how each cloud provider delivers those capabilities)
- **Workflow orchestration** (how capabilities are composed)
- **Business services** (domain-specific logic)
- **API exposure** (REST and MCP endpoints)

### Cloud Portability
While leveraging cloud-native AI services, the architecture avoids deep coupling to any single cloud platform. Deployments can target AWS, Azure, or GCP with configuration changes rather than code modifications.

### Dual Protocol Support
The platform exposes capabilities through both REST APIs (for traditional integrations) and MCP (Model Context Protocol) for direct AI assistant integration. Both protocols share the same underlying services.

## System Components

### AI/ML Core Layer

The foundational abstraction layer defines contracts that all AI providers must fulfill.

**Provider Interface**
Defines the core capabilities expected from any AI provider:
- Synchronous and asynchronous chat completions
- Tool/function calling support
- Text embedding generation
- Capability discovery and health checking

**Capability Model**
Not all providers support all features. The architecture uses capability markers to:
- Declare what each provider can do
- Enable runtime capability checking
- Support graceful degradation when capabilities are unavailable

**Common Models**
Shared data structures that abstract provider-specific formats:
- Chat messages and conversations
- Tool definitions and invocations
- Embedding vectors
- Token usage and billing information
- Standardized error representations

### Provider Implementations

Each supported AI platform has a dedicated implementation module.

**OpenAI Provider**
- Integration with OpenAI's chat and embedding APIs
- Support for GPT-4, GPT-4 Turbo, and future models
- Function calling with JSON schema validation
- Streaming response support

**Google Vertex AI Provider**
- Integration with Vertex AI's Gemini models
- Support for grounding and citation
- Google Cloud authentication handling
- Regional endpoint configuration

**AWS Bedrock Provider**
- Integration with Amazon Bedrock's model catalog
- Support for Claude, Titan, and other Bedrock models
- AWS credential chain integration
- Cross-region inference support

**Azure OpenAI Provider**
- Integration with Azure-hosted OpenAI models
- Azure Active Directory authentication
- Private endpoint support
- Deployment-based model selection

### Workflow Engine

Orchestrates complex AI operations across one or multiple providers.

**Execution Strategies**

*Sequential Execution*
Tasks execute one after another, with each task potentially using outputs from previous tasks. Useful for:
- Multi-step reasoning chains
- Progressive refinement of results
- Dependent operations where order matters

*Parallel Execution*
Multiple tasks execute simultaneously, with results aggregated upon completion. Useful for:
- Gathering responses from multiple providers
- Independent subtasks that can run concurrently
- Reducing total latency for complex operations

*Consensus Execution*
Parallel execution with result aggregation to determine agreement. Useful for:
- Reducing hallucination through cross-validation
- High-stakes decisions requiring multiple opinions
- Quality assurance on AI outputs

*Fallback Execution*
Sequential attempts across providers, stopping at first success. Useful for:
- High availability requirements
- Cost optimization (try cheaper providers first)
- Handling provider outages gracefully

*Race Execution*
Parallel execution returning the first completed response. Useful for:
- Latency-critical operations
- Redundancy without waiting for all responses

**Workflow Context**
Shared state that flows through workflow execution:
- Input parameters and configuration
- Intermediate results from completed tasks
- Accumulated token usage and costs
- Execution metadata and timing

**Task Aggregation**
Strategies for combining results from parallel executions:
- Majority voting for categorical outputs
- Weighted averaging for numerical outputs
- Union or intersection for list outputs
- Custom aggregation functions for domain-specific needs

### MCP Server Layer

Exposes platform capabilities through the Model Context Protocol for AI assistant integration.

**Tool Definitions**
Each business capability is exposed as an MCP tool with:
- Clear name and description for AI understanding
- JSON Schema parameter definitions
- Structured response formats

**Transport Support**
- Server-Sent Events (SSE) for real-time streaming
- HTTP-based communication suitable for cloud deployment
- Stateless design for horizontal scaling

**Integration Patterns**
- Direct integration with Claude Desktop and Claude.ai
- Support for custom AI agents as MCP clients
- Composable with other MCP servers

### REST API Layer

Traditional HTTP API for programmatic access and non-AI integrations.

**API Design**
- RESTful resource-oriented endpoints
- OpenAPI 3.0 specification
- Consistent error response format
- Pagination for list operations

**Documentation**
- Auto-generated Swagger UI
- Interactive API exploration
- Request/response examples

### Business Services

Domain-specific logic that both MCP tools and REST controllers consume.

**Inventory Service**
- Vehicle search with complex filtering
- Full-text search capabilities
- Result ranking and relevance scoring
- Inventory aggregation and statistics

**Financing Service**
- Payment calculation algorithms
- Interest rate estimation
- Amortization schedule generation
- Total cost projections

**Favorites Service**
- User-scoped vehicle bookmarking
- Favorites persistence and retrieval
- Change tracking for saved vehicles

**Filter Service**
- Dynamic filter option generation
- Count aggregation per filter value
- Filter dependency management

## Integration Patterns

### Pattern 1: Direct MCP Integration

AI assistants (Claude Desktop, Claude.ai) connect directly to the MCP server.

**Flow:**
1. User sends natural language query to AI assistant
2. AI assistant determines relevant tools to invoke
3. AI assistant calls MCP server tools
4. MCP server executes business logic
5. Results return to AI assistant
6. AI assistant formulates natural language response

**Characteristics:**
- Simplest integration path
- AI assistant handles conversation management
- Limited customization of AI behavior
- Suitable for development and demonstration

### Pattern 2: Custom Agent Integration

Custom AI agent orchestrates between user interface and platform capabilities.

**Flow:**
1. User sends query to custom chat interface
2. Custom backend receives query
3. Custom AI agent (using AI/ML core) processes query
4. Agent invokes MCP tools or REST APIs as needed
5. Agent formulates response using AI provider
6. Response returns to user through custom interface

**Characteristics:**
- Full control over AI behavior and prompting
- Custom conversation management
- Branded user experience
- Suitable for production deployments

### Pattern 3: Multi-Provider Orchestration

Complex queries leverage multiple AI providers through workflow engine.

**Flow:**
1. Query arrives requiring high-confidence response
2. Workflow engine distributes query to multiple providers
3. Providers respond independently and in parallel
4. Aggregation logic combines or validates responses
5. Consensus result returns to caller

**Characteristics:**
- Improved accuracy through cross-validation
- Higher latency due to parallel coordination
- Increased cost from multiple provider calls
- Suitable for high-stakes decisions

## Cross-Cutting Concerns

### Authentication and Authorization

**API Security**
- API key authentication for service-to-service communication
- OAuth 2.0 support for user-context operations
- Role-based access control for administrative functions

**Provider Credentials**
- Secure credential storage (environment variables, secrets managers)
- Credential rotation support
- Per-environment credential isolation

### Observability

**Logging**
- Structured JSON logging
- Correlation IDs across service boundaries
- Sensitive data redaction

**Metrics**
- Request latency histograms
- Provider response times
- Token usage tracking
- Error rates by category

**Tracing**
- Distributed tracing across components
- Provider call attribution
- Workflow execution visualization

### Resilience

**Circuit Breaking**
- Per-provider circuit breakers
- Automatic failure detection
- Gradual recovery with half-open state

**Rate Limiting**
- Provider rate limit awareness
- Request queuing and backpressure
- Fair scheduling across consumers

**Timeout Management**
- Configurable timeouts per operation
- Graceful degradation on timeout
- Partial result handling

### Configuration Management

**Environment-Based Configuration**
- Development, staging, production profiles
- Environment variable overrides
- Sensible defaults with explicit overrides

**Provider Configuration**
- Model selection per environment
- Endpoint URLs and regions
- Feature flags per provider

**Runtime Configuration**
- Dynamic configuration refresh
- Feature toggles without deployment
- A/B testing support

## Deployment Considerations

### Containerization
- Docker-based packaging
- Multi-stage builds for minimal images
- Health check endpoints for orchestration

### Scaling
- Stateless service design
- Horizontal scaling through replication
- Load balancer compatibility

### Cloud Deployment Options

**AWS**
- ECS or EKS for container orchestration
- Application Load Balancer for traffic distribution
- Secrets Manager for credentials
- CloudWatch for observability

**Azure**
- Azure Kubernetes Service or Container Apps
- Application Gateway for ingress
- Key Vault for secrets
- Application Insights for monitoring

**Google Cloud**
- Cloud Run or GKE for containers
- Cloud Load Balancing for traffic
- Secret Manager for credentials
- Cloud Operations for observability

## Data Flow Summary

### Search Request Flow

1. **Input**: Natural language query or structured search parameters
2. **Processing**: Query parsing, filter construction, inventory search
3. **Enrichment**: Price analysis, deal scoring, recommendation ranking
4. **Output**: Ranked vehicle list with explanatory metadata

### Financing Calculation Flow

1. **Input**: Vehicle price, down payment, term, credit profile
2. **Processing**: Interest rate determination, payment calculation
3. **Enrichment**: Amortization schedule, total cost projection
4. **Output**: Complete financing scenario with breakdown

### Workflow Execution Flow

1. **Input**: Workflow definition with tasks and strategy
2. **Dispatch**: Tasks distributed per execution strategy
3. **Execution**: Provider calls with context propagation
4. **Aggregation**: Results combined per aggregation strategy
5. **Output**: Unified result with execution metadata

## Technology Alignment

### Runtime Platform
- Java 17+ for long-term support and modern features
- Spring Boot 3.x for application framework
- Spring AI for AI provider abstractions
- Project Reactor for reactive programming

### Communication
- HTTP/REST for synchronous APIs
- Server-Sent Events for MCP streaming
- JSON for data serialization
- OpenAPI for API contracts

### Data Management
- JPA for relational data access
- H2 for development and testing
- Production database configurable (PostgreSQL, MySQL)

### Build and Dependency Management
- Maven for build orchestration
- BOM-based dependency management
- Multi-module project structure

### Quality and Testing
- JUnit 5 for unit testing
- Testcontainers for integration testing
- Contract testing for API stability

## Summary

The AutoMart AI Agents architecture delivers a flexible, maintainable, and cloud-portable platform for AI-powered vehicle search. Through careful abstraction of AI providers, robust workflow orchestration, and dual-protocol API exposure, the system balances immediate functionality with long-term adaptability. The modular design enables incremental adoption of new AI capabilities while protecting investments in business logic and integrations.