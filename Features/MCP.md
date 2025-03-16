# Model Context Protocol (MCP) - Enhancing Agent Capabilities

## Overview

Model Context Protocol (MCP) is a powerful mechanism in Alith that enables AI models to dynamically integrate with external services and data sources during inference. This protocol acts as a bridge between language models and various tools/APIs, allowing agents to access real-time information and execute complex operations without requiring custom code implementation.

> **🔌 Core Feature**: MCP transforms Alith agents into versatile systems that can seamlessly interact with external services like GitHub, databases, social media platforms, and custom APIs without modifying the agent's core code.

## Key Concepts

* **MCP Server**: An external process that implements the Model Context Protocol and provides tools to agents
* **MCP Client**: The component within Alith that communicates with MCP servers
* **Tool Discovery**: The mechanism by which agents automatically discover tools provided by MCP servers
* **Configuration File**: A JSON file that defines which MCP servers to connect to and how to start them
* **Process Isolation**: A security feature that runs each MCP server in its own separate process

## Usage

Setting up MCP is straightforward across all supported languages in Alith.

### Node.js

```javascript
import { Agent } from 'alith'

const agent = new Agent({
  name: 'GitHub Agent',
  model: 'gpt-4o',
  preamble: 'You are a helpful assistant that can interact with GitHub repositories.',
  mcpConfigPath: 'mcp_config.json',
})

console.log(agent.prompt('List my repositories'))
```

### Python

```python
from alith import Agent

agent = Agent(
    name="GitHub Agent",
    model="gpt-4o",
    preamble="You are a helpful assistant that can interact with GitHub repositories.",
    mcp_config_path="mcp_config.json",
)

print(agent.prompt("List my repositories"))
```

### Rust

```rust
use alith::{Agent, LLM};

#[tokio::main]
async fn main() -> Result<(), anyhow::Error> {
    let model = LLM::from_model_name("gpt-4o")?;
    let agent = Agent::new("GitHub Agent", model)
        .preamble("You are a helpful assistant that can interact with GitHub repositories.")
        .mcp_config_path("mcp_config.json").await?;
    
    let response = agent.prompt("List my repositories").await?;
    println!("{}", response);
    
    Ok(())
}
```

## Common Patterns

### Pattern 1: Integrating with External APIs

Use MCP to connect your agent with popular third-party services.

```javascript
// MCP configuration for GitHub integration
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"]
    }
  }
}
```

### Pattern 2: Custom Tool Integration

Create and integrate your own MCP servers with specialized tools.

```javascript
// MCP configuration for a custom API server
{
  "mcpServers": {
    "customApi": {
      "command": "node",
      "args": ["./my-custom-mcp-server.js"],
      "env": {
        "API_KEY": "your-api-key"
      }
    }
  }
}
```

## Configuration Options

| Option | Type | Description |
|--------|------|-------------|
| `mcpServers` | object | Map of server names to server configurations |
| `mcpServers.*.command` | string | The command to execute the server (e.g., `npx`, `python`) |
| `mcpServers.*.args` | array | Command line arguments to pass to the server |
| `mcpServers.*.env` | object | Environment variables to set for the server process |
| `mcpServers.*.cwd` | string | Working directory for the server process (optional) |

## Best Practices

* **Version Control**: Specify exact versions of MCP servers in your configuration to ensure consistency
* **Authentication**: Use environment variables in the configuration to pass sensitive information like API keys
* **Modularity**: Create separate MCP servers for different services to maintain clean separation of concerns
* **Error Handling**: Configure timeouts and validate server responses to handle potential failures gracefully
* **Documentation**: Document the available tools provided by each MCP server for developers using your agent

## Common Issues and Solutions

### Issue 1: MCP Server Not Starting

When the agent fails to connect to an MCP server.

**Solution**: Verify the command and arguments in your configuration. Make sure the required packages are installed globally or locally as needed.

```bash
# For Node.js MCP servers
npm install -g @modelcontextprotocol/server-github

# For Python MCP servers
pip install mcp-server-example
```

### Issue 2: Tools Not Discovered

When the agent doesn't recognize tools that should be available.

**Solution**: Check server logs for errors and ensure the server implements the tool discovery protocol correctly.

## Advanced Usage

For developers who want to create their own MCP servers:

```javascript
// Example of a simple MCP server in Node.js
const { createServer } = require('@modelcontextprotocol/server');

const server = createServer();

server.registerTool({
  name: 'getCurrentWeather',
  description: 'Get the current weather for a location',
  inputSchema: {
    type: 'object',
    properties: {
      location: {
        type: 'string',
        description: 'The city and state, e.g. San Francisco, CA'
      }
    },
    required: ['location']
  },
  handler: async ({ location }) => {
    // Implementation to fetch weather data
    return { temperature: 72, conditions: 'sunny' };
  }
});

server.start();
```

## Related Documentation

* [Tools](Tools.md) - Learn about Alith's native tool system that complements MCP
* [Knowledge](Knowledge.md) - Combine MCP with knowledge bases for more context-aware responses
* [Memory](Memory.md) - Enable agents to remember context across sessions with external APIs

## API Reference

```typescript
// TypeScript interface for MCP configuration
interface McpServerConfig {
  command: string;
  args: string[];
  env?: Record<string, string>;
  cwd?: string;
}

interface McpConfig {
  mcpServers: Record<string, McpServerConfig>;
}

// Agent configuration with MCP
interface AgentOptions {
  name: string;
  model: string;
  preamble?: string;
  mcpConfigPath?: string;
  // Other agent options...
}
```

---

*Last Updated: March 17, 2024* 