# Claude-Flow Orchestrator Prompts & Strategy Analysis

## Overview
This document contains all prompts, strategies, and workflow planning details extracted from the claude-flow orchestrator codebase.

---

## 1. INTENT DETECTION & COMPLEXITY ESTIMATION

### Location: `src/swarm/strategies/auto.ts`

The Auto Strategy uses ML-inspired heuristics for intelligent request analysis:

#### **Pattern Detection** (Async)
Detects task patterns using regex and content analysis with caching.

```typescript
private async detectPatternsAsync(description: string): Promise<TaskPattern[]>
```

#### **Task Type Analysis** (Lines 247-275)
Categorizes user requests into specific task types:

```typescript
// Task Type Detection Keywords:
if (/create|build|implement|develop|code/i.test(description)) → 'development'
if (/test|verify|validate|check/i.test(description)) → 'testing'
if (/analyze|research|investigate|study/i.test(description)) → 'analysis'
if (/document|write|explain|describe/i.test(description)) → 'documentation'
if (/optimize|improve|enhance|refactor/i.test(description)) → 'optimization'
if (/deploy|install|configure|setup/i.test(description)) → 'deployment'
```

#### **Complexity Estimation** (Lines 277-292)
Calculates complexity with ML heuristic weights:

```typescript
ML Complexity Factors:
{
  integration: 1.5,
  system: 1.3,
  api: 1.2,
  database: 1.4,
  ui: 1.1,
  algorithm: 1.6
}

// Final complexity: Min(Round(baseComplexity * factors), 5)
```

#### **Task Type Weights** (Lines 198-206)
```typescript
taskTypeWeights: {
  development: 1.0,
  testing: 0.8,
  analysis: 0.9,
  documentation: 0.6,
  optimization: 1.1,
  research: 0.7
}
```

---

## 2. WORKFLOW PLANNING & DECOMPOSITION

### Location: `src/swarm/coordinator.ts:1578-1758`

The `decomposeObjective()` method analyzes the user request and generates structured task workflows.

---

## 3. STRATEGY-SPECIFIC PROMPTS

### 3.1 DEVELOPMENT STRATEGY
**Location**: `src/swarm/coordinator.ts:1631-1724`

Creates a 4-phase workflow:

#### **Phase 1: Analysis & Planning** (Task 1)
```
Please analyze the following request and create a detailed implementation plan:

Request: ${objective.description}

Target Directory: ${targetPath || 'Not specified - determine appropriate location'}

Your analysis should include:
1. Understanding of what needs to be built
2. Technology choices and rationale
3. Project structure and file organization
4. Key components and their responsibilities
5. Any external dependencies needed

Please provide a clear, structured plan that the next tasks can follow.
```
- **Priority**: High
- **Duration**: 5 minutes
- **Capabilities**: analysis, documentation

#### **Phase 2: Implementation** (Task 2)
```
Please implement the following request:

Request: ${objective.description}

Target Directory: ${targetPath || 'Create in an appropriate location'}

Based on the analysis from the previous task, please:
1. Create all necessary files and directories
2. Implement the core functionality as requested
3. Ensure the code is well-structured and follows best practices
4. Include appropriate error handling
5. Add any necessary configuration files (package.json, requirements.txt, etc.)

Focus on creating a working implementation that matches the user's request exactly.
```
- **Priority**: High
- **Duration**: 10 minutes
- **Capabilities**: code-generation, file-system
- **Dependencies**: Task 1

#### **Phase 3: Testing** (Task 3)
```
Please create comprehensive tests for the implementation created in the previous task.

Target Directory: ${targetPath || 'Use the same directory as the implementation'}

Create appropriate test files that:
1. Test the main functionality
2. Cover edge cases
3. Ensure the implementation works as expected
4. Use appropriate testing frameworks for the technology stack
5. Include both unit tests and integration tests where applicable
```
- **Priority**: Medium
- **Duration**: 5 minutes
- **Capabilities**: testing, code-generation
- **Dependencies**: Task 2

#### **Phase 4: Documentation** (Task 4)
```
Please create comprehensive documentation for the implemented solution.

Target Directory: ${targetPath || 'Use the same directory as the implementation'}

Create documentation that includes:
1. README.md with project overview, setup instructions, and usage examples
2. API documentation (if applicable)
3. Configuration options
4. Architecture overview
5. Deployment instructions (if applicable)
6. Any other relevant documentation

Make sure the documentation is clear, complete, and helps users understand and use the implementation.
```
- **Priority**: Medium
- **Duration**: 5 minutes
- **Capabilities**: documentation
- **Dependencies**: Task 2

---

### 3.2 RESEARCH STRATEGY
**Location**: `src/swarm/strategies/research.ts:218-400`

Creates a 5-phase parallel workflow:

#### **Phase 1: Query Planning**
```
Analyze the research objective and create optimized search queries:

${objective.description}

Create a comprehensive research plan that includes:
1. Primary and secondary research questions
2. Key search terms and synonyms
3. Relevant domains and sources to explore
4. Research methodology and approach
5. Quality criteria for evaluating sources

Focus on creating queries that will yield high-quality, credible results.
```
- **Priority**: High
- **Duration**: 5 minutes
- **Capabilities**: research, analysis

#### **Phase 2: Parallel Web Search**
```
Execute parallel web searches based on the research plan:

${objective.description}

Perform comprehensive web searches using:
1. Multiple search engines and sources
2. Parallel query execution for efficiency
3. Intelligent source ranking and filtering
4. Real-time credibility assessment
5. Deduplication of results

Collect diverse, high-quality sources relevant to the research objective.
```
- **Priority**: High
- **Duration**: 10 minutes
- **Capabilities**: web-search, research
- **Dependencies**: Phase 1

#### **Phase 3: Parallel Data Extraction**
```
Extract and process data from collected sources:

${objective.description}

Process the collected sources by:
1. Extracting key information and insights
2. Performing semantic analysis and clustering
3. Identifying patterns and relationships
4. Assessing information quality and reliability
5. Creating structured summaries

Use parallel processing for efficient data extraction.
```
- **Priority**: High
- **Duration**: 8 minutes
- **Capabilities**: analysis, research
- **Dependencies**: Phase 2

#### **Phase 4: Semantic Clustering**
```
Perform semantic clustering of research findings:

${objective.description}

Analyze the extracted data by:
1. Grouping related information using semantic similarity
2. Identifying key themes and topics
3. Creating coherent clusters of information
4. Generating cluster summaries and insights
5. Mapping relationships between clusters

Provide a structured analysis of the research findings.
```
- **Priority**: Medium
- **Duration**: 6 minutes
- **Capabilities**: analysis, research
- **Dependencies**: Phase 3

#### **Phase 5: Synthesis & Reporting**
```
Synthesize research findings into comprehensive report:

${objective.description}

Create a comprehensive research report that includes:
1. Executive summary of key findings
2. Detailed analysis of each research cluster
3. Insights and recommendations
4. Source credibility assessment
5. Methodology and limitations
6. References and citations

Ensure the report is well-structured and actionable.
```
- **Priority**: Medium
- **Duration**: 7 minutes
- **Capabilities**: documentation, analysis
- **Dependencies**: Phase 4

---

### 3.3 GENERIC / AUTO STRATEGY
**Location**: `src/swarm/coordinator.ts:1726-1749`

Creates a comprehensive single-task workflow:

```
Please complete the following request:

${objective.description}

${targetPath ? `Target Directory: ${targetPath}` : ''}

Please analyze what is being requested and implement it appropriately. This may involve:
- Creating files and directories
- Writing code
- Setting up configurations
- Creating documentation
- Any other tasks necessary to fulfill the request

Ensure your implementation is complete, well-structured, and follows best practices.
```
- **Priority**: High
- **Duration**: 15 minutes
- **Capabilities**: code-generation, file-system, documentation

---

### 3.4 PARALLEL AGENTS STRATEGY
**Location**: `src/swarm/coordinator.ts:1603-1630`

When user requests "each agent" or "each agent type", creates parallel tasks for multiple agent types:

```
You are a ${agentType} agent. Please execute the following task from your perspective:

${objective.description}

${targetPath ? `Target Directory: ${targetPath}` : ''}

As a ${agentType} agent, focus on aspects relevant to your role:
${this.getAgentTypeInstructions(agentType)}

Work independently but be aware that other agents are working on this same objective from their perspectives.
```

---

## 4. AGENT TYPE INSTRUCTIONS

### Location: `src/swarm/coordinator.ts:1536-1555`

Specific instructions tailored to each agent type:

#### **Coder Agent**
```
- Focus on implementation, code quality, and best practices
- Create clean, maintainable code
- Consider architecture and design patterns
```
**Capabilities**: code-generation, file-system, debugging

#### **Tester Agent**
```
- Focus on testing, edge cases, and quality assurance
- Create comprehensive test suites
- Identify potential bugs and issues
```
**Capabilities**: testing, code-generation, analysis

#### **Analyst Agent**
```
- Focus on analysis, research, and understanding
- Break down complex problems
- Provide insights and recommendations
```
**Capabilities**: analysis, documentation, research

#### **Researcher Agent**
```
- Focus on gathering information and best practices
- Research existing solutions and patterns
- Document findings and recommendations
```
**Capabilities**: research, documentation, analysis

#### **Reviewer Agent**
```
- Focus on code review and quality checks
- Identify improvements and optimizations
- Ensure standards compliance
```
**Capabilities**: code-review, analysis, documentation

#### **Coordinator Agent**
```
- Focus on coordination and integration
- Ensure all parts work together
- Manage dependencies and interfaces
```
**Capabilities**: coordination, analysis, documentation

#### **Monitor Agent**
```
- Focus on monitoring and observability
- Set up logging and metrics
- Ensure system health tracking
```
**Capabilities**: monitoring, analysis, documentation

---

## 5. ORCHESTRATION WITH DEPENDENCIES & PARALLEL PROCESSING

### Dependency Management
**Location**: `src/swarm/coordinator.ts`, `src/task/engine.ts`

The orchestrator supports 4 dependency types:

1. **finish-to-start**: Task starts only after dependency completes
2. **start-to-start**: Task starts when dependency starts
3. **finish-to-finish**: Task finishes when dependency finishes
4. **start-to-finish**: Task finishes when dependency starts

### Parallel Processing Strategy

#### **Batch Groups** (Lines 127-129 in auto.ts)
```typescript
const dependencies = this.analyzeDependencies(tasks);
const batchGroups = this.createTaskBatches(tasks, dependencies);
const estimatedDuration = this.calculateOptimizedDuration(batchGroups);
```

Tasks are grouped into batches that can run in parallel:
- Tasks with no dependencies run immediately
- Tasks are scheduled as soon as all dependencies are satisfied
- Maximum concurrent tasks configurable (default: 10)

#### **Parallel Execution Context** (`src/swarm/advanced-orchestrator.ts`)
```typescript
parallelism: {
  maxConcurrent: number,
  strategy: 'breadth-first' | 'depth-first' | 'priority-based'
}
```

#### **Task Scheduling Algorithm**
1. Analyze task dependencies
2. Create dependency graph
3. Identify tasks with no dependencies (ready queue)
4. Execute ready tasks in parallel (up to maxConcurrent limit)
5. As tasks complete, check dependents for readiness
6. Continue until all tasks complete

### Resource Allocation

The orchestrator manages:
- **Agent allocation**: Assigns agents based on capabilities and workload
- **Resource pooling**: Connection pooling, memory pooling
- **Load balancing**: Work-stealing, adaptive scheduling
- **Rate limiting**: Per-domain rate limits with exponential backoff

---

## 6. STRATEGY DETECTION

### Available Strategies
**Location**: `src/swarm/coordinator.ts:1517-1534`

```typescript
private determineRequiredAgentTypes(strategy: SwarmStrategy): AgentType[] {
  switch (strategy) {
    case 'development':
      return ['coder', 'tester', 'reviewer'];
    case 'research':
      return ['researcher', 'analyst'];
    case 'analysis':
      return ['analyst', 'researcher'];
    case 'testing':
      return ['tester', 'coder'];
    case 'optimization':
      return ['analyst', 'coder'];
    case 'maintenance':
      return ['coder', 'monitor'];
    default:
      return ['coordinator', 'coder', 'analyst'];
  }
}
```

**Summary of Strategies:**

| Strategy | Required Agent Types |
|----------|---------------------|
| **development** | coder, tester, reviewer |
| **research** | researcher, analyst |
| **analysis** | analyst, researcher |
| **testing** | tester, coder |
| **optimization** | analyst, coder |
| **maintenance** | coder, monitor |
| **default/auto** | coordinator, coder, analyst |

### Strategy Selection (Auto Strategy)
**Location**: `src/swarm/strategies/auto.ts:102-153`

The Auto strategy automatically selects the optimal strategy based on:
1. Detected patterns in the user request
2. Task type analysis (development, testing, analysis, etc.)
3. Complexity estimation
4. ML-inspired heuristics

---

## 7. RESEARCH STRATEGY OPTIMIZATIONS

The Research strategy includes advanced optimizations:

### **Caching System**
- Cache key generation based on query content
- TTL-based expiration (1-2 hours)
- LRU eviction when cache size > 1000 entries

### **Connection Pooling**
```typescript
connectionPool: {
  max: 10,
  timeout: 30000,
  active: number,
  idle: number
}
```

### **Rate Limiting**
- Per-domain rate limiters
- 10 requests per 60-second window
- Exponential backoff on limit exceeded

### **Parallel Processing**
- Parallel query execution
- Batch data extraction
- Concurrent semantic analysis

### **Credibility Scoring**
```typescript
combinedScore = credibilityScore * 0.6 + relevanceScore * 0.4
```

---

## 8. EXTENDED TASK TYPES

### Auto Strategy Extended Types
**Location**: `src/swarm/strategies/auto.ts:1-42`

```typescript
Extended Task Types:
- data-analysis, performance-analysis, statistical-analysis
- visualization, predictive-modeling, anomaly-detection
- trend-analysis, business-intelligence, quality-analysis
- system-design, architecture-review, api-design
- cloud-architecture, microservices-design, security-architecture
- scalability-design, database-architecture
- code-generation, code-review, refactoring, debugging
- api-development, database-design, performance-optimization
- task-orchestration, progress-tracking, resource-allocation
- workflow-management, team-coordination, status-reporting
- fact-check, literature-review, market-analysis
- unit-testing, integration-testing, e2e-testing
- performance-testing, security-testing, api-testing
- test-automation, test-analysis
```

---

## Summary

The claude-flow orchestrator uses a sophisticated multi-layered approach:

1. **Intent Detection**: ML-inspired pattern detection and complexity estimation
2. **Strategy Selection**: Auto-selects optimal strategy (development, research, analysis, etc.)
3. **Workflow Decomposition**: Breaks down objectives into sequential and parallel tasks
4. **Agent Assignment**: Matches agents to tasks based on capabilities and workload
5. **Parallel Execution**: Executes independent tasks concurrently with dependency management
6. **Resource Optimization**: Connection pooling, caching, rate limiting, load balancing
7. **Quality Control**: Built-in testing, review, and documentation phases

Each strategy provides tailored prompts and instructions optimized for specific use cases, enabling efficient multi-agent collaboration at scale.
