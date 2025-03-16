# Eliza Integration

## Overview

The Eliza integration allows you to connect Alith with ElizaOS, enabling powerful AI agent capabilities across both ecosystems. This integration offers bidirectional compatibility: you can use Alith agents within ElizaOS or leverage ElizaOS plugins within Alith. This flexibility allows you to choose the best approach for your specific use case while maintaining high-performance AI capabilities.

> **🔗 Integration Feature**: Combining Alith's high-performance agent capabilities with ElizaOS's plugin ecosystem gives you the best of both worlds for building sophisticated AI applications.

## Key Concepts

* **Alith Plugin for ElizaOS**: Use Alith agents within ElizaOS for enhanced capabilities
* **ElizaOS Plugins in Alith**: Leverage existing ElizaOS plugins in your Alith applications
* **Multi-Agent Interaction**: Enable communication between agents across both ecosystems
* **Template-Free Development**: Use Alith's structured tools without complex template engineering
* **Performance Optimization**: Benefit from Alith's high-performance inference capabilities

## Integration Methods

You can integrate Alith and ElizaOS in two primary ways:

### Method 1: Enable Alith Plugin in ElizaOS

This approach allows you to use Alith agents within the ElizaOS ecosystem. Key benefits include:

- Multi-agent interaction by passing Alith agent instances to ElizaOS
- Using Alith's advanced features (tools, extractors) within ElizaOS plugins
- Template-free development for complex plugins
- High-performance inference optimization

```javascript
import { createAlithPlugin, Agent } from "elizaos-alith";
import {
	AgentRuntime,
	CacheManager,
	MemoryCacheAdapter,
	ModelProviderName,
} from "@elizaos/core";
 
// Initialize an Alith agent
const provider = ModelProviderName.OPENAI;
const agent = new Agent({
	name: "ComedianAgent",
	model: "gpt-4",
	preamble: "You are a comedian here to entertain the user using humor and jokes.",
	provider,
});

// Set up ElizaOS runtime with Alith plugin
const runtime = new AgentRuntime({
	token: "your-api-token",
	databaseAdapter: new YourDatabaseAdapter(),
	cacheManager: new CacheManager(new MemoryCacheAdapter()),
	modelProvider: provider,
	// Add Alith plugin to the ElizaOS agent runtime
	plugins: [
		createAlithPlugin(agent),
		// Add other ElizaOS plugins as needed
	],
});

// Now you can use the runtime with Alith capabilities
```

### Method 2: Use ElizaOS Plugins within Alith

This approach allows you to leverage existing ElizaOS plugins within your Alith applications. Key benefits include:

- Access to the wide variety of plugins from the ElizaOS ecosystem
- Using familiar ElizaOS plugins without rewriting them
- Maintaining Alith's core performance advantages

```javascript
import { Agent } from "elizaos-alith";
import { ModelProviderName } from "@elizaos/core";
 
// Initialize Alith agent with ElizaOS plugins
const provider = ModelProviderName.OPENAI;
const agent = new Agent({
	name: "ComedianAgent",
	model: "gpt-4",
	preamble: "You are a comedian here to entertain the user using humor and jokes.",
	provider,
	plugins: [
		// Add your ElizaOS plugins here
		elizaPlugin1,
		elizaPlugin2
	]
});

// Use the agent with ElizaOS plugins
const result = await agent.prompt("Tell me a joke about programming");
console.log(result);
```

## Example: Advanced Multi-Agent System

Here's a more complex example that combines both integration methods to create a multi-agent system:

```javascript
import { createAlithPlugin, Agent } from "elizaos-alith";
import {
	AgentRuntime,
	CacheManager,
	MemoryCacheAdapter,
	ModelProviderName,
	ElizaPlugin
} from "@elizaos/core";

// Create specialized Alith agents
const researchAgent = new Agent({
	name: "ResearchAgent",
	model: "gpt-4",
	preamble: "You are a research assistant who finds detailed information.",
	provider: ModelProviderName.OPENAI,
	tools: [webSearchTool, databaseQueryTool]
});

const summaryAgent = new Agent({
	name: "SummaryAgent",
	model: "gpt-3.5-turbo",
	preamble: "You are an expert at summarizing complex information concisely.",
	provider: ModelProviderName.OPENAI
});

// Create ElizaOS plugins using Alith agents
const researchPlugin = createAlithPlugin(researchAgent);
const summaryPlugin = createAlithPlugin(summaryAgent);

// Create an ElizaOS custom plugin
const customElizaPlugin = new ElizaPlugin({
	name: "data-formatter",
	description: "Formats data into structured outputs",
	// Plugin implementation
});

// Create an Alith agent that uses the ElizaOS plugin
const mainAgent = new Agent({
	name: "OrchestrationAgent",
	model: "gpt-4",
	preamble: "You coordinate research and summarization tasks.",
	provider: ModelProviderName.OPENAI,
	plugins: [customElizaPlugin]
});

// Set up the ElizaOS runtime with all components
const runtime = new AgentRuntime({
	token: "your-api-token",
	databaseAdapter: new YourDatabaseAdapter(),
	cacheManager: new CacheManager(new MemoryCacheAdapter()),
	modelProvider: ModelProviderName.OPENAI,
	plugins: [
		researchPlugin,
		summaryPlugin,
		createAlithPlugin(mainAgent)
	],
});

// Now you can use this complex multi-agent system
```

## Configuration Options

| Option | Type | Description |
|--------|------|-------------|
| `model` | string | Language model to use (e.g., "gpt-4") |
| `provider` | ModelProviderName | The model provider (e.g., OPENAI, ANTHROPIC) |
| `plugins` | Array | Array of ElizaOS plugins to use with Alith |
| `token` | string | API token for authentication |
| `cacheManager` | CacheManager | Cache implementation for optimization |
| `databaseAdapter` | DatabaseAdapter | Adapter for persistent storage |

## Best Practices

* **Choose the Right Integration**: Use Method 1 when you need Alith's advanced capabilities in ElizaOS; use Method 2 when you want to leverage existing ElizaOS plugins
* **Optimize Performance**: Utilize Alith's performance optimizations for intensive workloads
* **Combine Approaches**: For complex applications, consider using both methods to maximize flexibility
* **Template-Free Development**: Leverage Alith's structured tools to avoid complex template engineering
* **Plugin Management**: Keep track of plugin dependencies and versions across both ecosystems

> **💡 Tip:** When working with multiple agents across ecosystems, establish clear communication patterns and responsibility boundaries to avoid conflicting behaviors.

## Common Issues and Solutions

### Issue 1: Plugin Compatibility

Some ElizaOS plugins may require specific versions or have dependencies that conflict with Alith.

**Solution**: Isolate plugin dependencies and use version management:

```javascript
// Isolate plugin dependencies
const pluginWithIsolatedDeps = isolatePluginDependencies(elizaPlugin, {
	forceVersion: {
		"dependency-name": "1.2.3"
	}
});

const agent = new Agent({
	// ...configuration
	plugins: [pluginWithIsolatedDeps]
});
```

### Issue 2: Performance Bottlenecks

Using multiple plugins can lead to performance issues.

**Solution**: Implement caching and batching:

```javascript
import { CacheManager, BatchProcessor } from "@elizaos/core";

// Set up caching
const cache = new CacheManager({
	adapter: new RedisCacheAdapter(),
	ttl: 3600 // 1 hour cache
});

// Set up batching for similar requests
const batchProcessor = new BatchProcessor({
	maxBatchSize: 5,
	timeoutMs: 200
});

const runtime = new AgentRuntime({
	// ...configuration
	cacheManager: cache,
	batchProcessor: batchProcessor
});
```

## Related Documentation

* [Web3Integration.md](Web3Integration.md) - Combine ElizaOS integration with blockchain capabilities
* [Langchain.md](Langchain.md) - Compare with alternative integration options
* [Features/Tools.md](../Features/Tools.md) - Learn about Alith tools that can be used with ElizaOS
* [Tutorials/MultiAgentSystem.md](../Tutorials/MultiAgentSystem.md) - See a complete multi-agent system example
* [Developing/NodeJs.md](../Developing/NodeJs.md) - Node.js development environment setup

## API Reference

```typescript
// Alith Plugin for ElizaOS
interface AlithPluginOptions {
	agent: Agent;
	name?: string;
	description?: string;
	fallback?: boolean;
}

function createAlithPlugin(agent: Agent, options?: Partial<AlithPluginOptions>): ElizaPlugin;

// ElizaOS Plugin Usage in Alith
interface ElizaPluginConfig {
	name: string;
	description: string;
	version?: string;
	author?: string;
	handlers: Record<string, PluginHandler>;
}

// ElizaOS Runtime Configuration
interface AgentRuntimeConfig {
	token: string;
	modelProvider: ModelProviderName;
	databaseAdapter: DatabaseAdapter;
	cacheManager?: CacheManager;
	plugins?: ElizaPlugin[];
}
```

---

*Last Updated: 2025-03-16*