# Getting Started with Alith

## Overview

This guide will walk you through the basics of setting up and using the Alith framework to build AI agents with blockchain capabilities. You'll learn how to install Alith in your preferred language, create your first agent, add custom tools, implement memory, and integrate with blockchain data.

> **🚀 Quick Start**: Follow these instructions to set up an Alith project in minutes, or explore the [learning paths](LearningPaths.md) for a structured approach to mastering Alith.

## Key Concepts

* **Agent**: The core entity that interacts with AI models to process prompts and generate responses
* **Tools**: Functions that extend agent capabilities by allowing them to perform specific tasks
* **Memory**: Systems that enable agents to remember previous interactions and maintain context
* **Preamble**: Instructions that define the agent's personality and how it should respond
* **Web3 Integration**: Capabilities for interacting with blockchain data and smart contracts

## Installation

Alith provides SDKs for multiple programming languages. Choose the one that best fits your project:

### Node.js

```bash
$ npm install alith
# Expected output:
+ alith@0.4.2
added 125 packages in 3.5s
```

### Python

```bash
$ pip install alith
# Expected output:
Successfully installed alith-0.4.2 ...
```

### Rust

Add to your `Cargo.toml`:

```toml
[dependencies]
alith = "0.4.2"
```

Then run:

```bash
$ cargo build
```

> **⚠️ Warning:** Make sure to use Alith version 0.4.2 or newer to access all features described in this documentation.

## Setup Environment

Most Alith applications require API keys for AI models and possibly blockchain providers:

1. Create a `.env` file in your project root
2. Add your API keys:

```
# AI Model Keys
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_anthropic_key_here

# Optional: Blockchain Provider Keys
INFURA_API_KEY=your_infura_key_here
ALCHEMY_API_KEY=your_alchemy_key_here
```

3. Load environment variables in your application:

### Node.js

```javascript
// Load environment variables
require('dotenv').config();
```

### Python

```python
# Load environment variables
from dotenv import load_dotenv
load_dotenv()
```

### Rust

```rust
// Load environment variables
use dotenv::dotenv;
dotenv().ok();
```

> **💡 Tip:** Never hardcode API keys in your application code. Always use environment variables or secure key management solutions.

## Creating a Basic Agent

Let's start with a simple agent that can respond to user prompts:

### Node.js Example

```javascript
const { Agent } = require('alith');

// Create a basic agent
const agent = new Agent({
  name: "HelloAlith",
  model: "gpt-3.5-turbo", // or any supported model
  preamble: "You are a helpful assistant."
});

// Use the agent
async function main() {
  const response = await agent.prompt("Tell me about blockchain technology");
  console.log(response);
}

main();
```

### Python Example

```python
from alith import Agent

# Create a basic agent
agent = Agent(
    name="HelloAlith",
    model="gpt-3.5-turbo",
    preamble="You are a helpful assistant."
)

# Use the agent
response = agent.prompt("Tell me about blockchain technology")
print(response)
```

### Rust Example

```rust
use alith::{Agent, LLM};

#[tokio::main]
async fn main() -> Result<(), anyhow::Error> {
    // Create a model instance
    let model = LLM::from_model_name("gpt-3.5-turbo")?;
    
    // Create a basic agent
    let agent = Agent::new("HelloAlith", model, vec![])
        .preamble("You are a helpful assistant.");
    
    // Use the agent
    let response = agent.prompt("Tell me about blockchain technology").await?;
    println!("{}", response);
    
    Ok(())
}
```

## Common Patterns

### Pattern 1: Adding Tools to Your Agent

Tools allow your agent to perform actions and access external systems:

```javascript
const { Agent, Tool } = require('alith');

// Define a simple calculator tool
const calculatorTool = new Tool({
  name: "calculator",
  description: "Perform a mathematical calculation",
  schema: {
    type: "object",
    properties: {
      expression: {
        type: "string",
        description: "The mathematical expression to evaluate"
      }
    },
    required: ["expression"]
  },
  function: async (params) => {
    try {
      // Simple evaluation (use a safer method in production)
      const result = eval(params.expression);
      return `The result is: ${result}`;
    } catch (error) {
      return `Error: ${error.message}`;
    }
  }
});

// Create an agent with the calculator tool
const agent = new Agent({
  name: "ToolAgent",
  model: "gpt-4", // GPT-4 is better at using tools
  preamble: "You are a helpful assistant that can perform calculations.",
  tools: [calculatorTool]
});

// Use the agent with the tool
async function main() {
  const response = await agent.prompt("What is 123 * 456?");
  console.log(response);
}

main();
```

> **See Also:** For more advanced tool usage, check out the [Tools documentation](Features/Tools.md).

### Pattern 2: Adding Memory to Your Agent

Memory allows your agent to remember previous interactions:

```javascript
const { Agent } = require('alith');

// Create an agent with memory
const agent = new Agent({
  name: "MemoryAgent",
  model: "gpt-4",
  preamble: "You are a helpful assistant with memory.",
  memory: {
    type: "buffer",
    capacity: 5 // Remember last 5 interactions
  }
});

// Demonstrate memory capabilities
async function conversationDemo() {
  console.log(await agent.prompt("Hello, my name is Alice."));
  console.log(await agent.prompt("What's my name?"));
  console.log(await agent.prompt("Tell me about blockchain."));
  console.log(await agent.prompt("What was the last topic we discussed?"));
}

conversationDemo();
```

> **See Also:** Explore more memory systems in the [Memory documentation](Features/Memory.md).

### Pattern 3: Web3 Integration Example

Connect your agent to blockchain data:

```javascript
const { Agent, Tool } = require('alith');
const { ethers } = require('ethers');

// Create an Ethereum provider
const provider = new ethers.providers.JsonRpcProvider(
  process.env.ETHEREUM_RPC_URL
);

// Define a blockchain balance checking tool
const balanceChecker = new Tool({
  name: "check_eth_balance",
  description: "Check the ETH balance of an Ethereum address",
  schema: {
    type: "object",
    properties: {
      address: {
        type: "string",
        description: "Ethereum address to check"
      }
    },
    required: ["address"]
  },
  function: async (params) => {
    try {
      const balance = await provider.getBalance(params.address);
      return `Balance: ${ethers.utils.formatEther(balance)} ETH`;
    } catch (error) {
      return `Error: ${error.message}`;
    }
  }
});

// Create an agent with the blockchain tool
const agent = new Agent({
  name: "Web3Agent",
  model: "gpt-4",
  preamble: "You are a blockchain-aware assistant that can check Ethereum balances.",
  tools: [balanceChecker]
});

// Use the agent to check a balance
async function main() {
  const response = await agent.prompt("What is the ETH balance of 0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045?");
  console.log(response);
}

main();
```

> **See Also:** Learn more about blockchain integration in [Web3 Integration](Integrations/Web3Integration.md).

## Configuration Options

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `name` | string | `'AlithAgent'` | Friendly name for your agent |
| `model` | string | `'gpt-3.5-turbo'` | The AI model to use (e.g., 'gpt-4', 'claude-2') |
| `preamble` | string | `''` | System prompt/instructions for the agent |
| `tools` | array | `[]` | List of tools available to the agent |
| `memory` | object | `null` | Memory configuration for the agent |
| `timeout` | number | `30000` | Timeout for agent operations in milliseconds |
| `maxRetries` | number | `3` | Number of retry attempts for failed requests |
| `streaming` | boolean | `false` | Whether to stream responses (real-time output) |

## Best Practices

* **Right-Size Models**: Use simpler models (GPT-3.5) for basic tasks and more powerful models (GPT-4, Claude) for complex reasoning or tool use
* **Clear Preambles**: Write specific and concise instructions in your agent's preamble
* **Tool Design**: Create tools with descriptive names and clear parameter descriptions
* **Memory Capacity**: Balance memory capacity with token limits of your chosen model
* **Error Handling**: Implement robust error handling in your tools and agent interactions
* **Environment Segregation**: Keep API keys and sensitive data in environment variables

> **📝 Note:** The guidelines above represent general best practices based on common usage patterns. For specific use cases, you may need to adjust these recommendations.

## Common Issues and Solutions

### Issue 1: Agent Not Using Tools

The agent sometimes ignores available tools and tries to answer directly.

**Solution**: Improve the tool descriptions and add explicit instructions in the preamble:

```javascript
const agent = new Agent({
  // ...other configuration
  preamble: "You are a helpful assistant with access to tools. ALWAYS use the appropriate tool when you need to perform a calculation or access external data. Do not attempt to perform these operations yourself."
});
```

### Issue 2: Memory Token Limits

When using memory, you may encounter token limit issues with the AI model.

**Solution**: Limit memory capacity and implement summarization:

```javascript
const agent = new Agent({
  // ...other configuration
  memory: {
    type: "summarizing",
    capacity: 10,
    summarizeAfter: 8,  // Summarize when we reach 8 messages
    tokenLimit: 3000    // Maximum tokens for memory context
  }
});
```

### Issue 3: Rate Limiting

You might encounter rate limit errors when making many requests to AI providers.

**Solution**: Implement exponential backoff and request throttling:

```javascript
const { Agent, createRateLimiter } = require('alith');

// Create a rate limiter
const rateLimiter = createRateLimiter({
  maxRequests: 60,     // 60 requests
  perTimeWindow: 60000 // per minute (60000ms)
});

// Create an agent with rate limiting
const agent = new Agent({
  // ...other configuration
  rateLimiter: rateLimiter
});

// For manual handling
async function safePrompt(agent, prompt) {
  try {
    return await agent.prompt(prompt);
  } catch (error) {
    if (error.code === 'rate_limit_exceeded') {
      console.log('Rate limited, retrying after delay...');
      await new Promise(resolve => setTimeout(resolve, 5000));
      return safePrompt(agent, prompt);
    }
    throw error;
  }
}
```

## Advanced Usage

For more complex applications, you can combine multiple agents with different capabilities:

```javascript
const { Agent, Process } = require('alith');

// Create specialized agents
const researchAgent = new Agent({
  name: "Researcher",
  model: "gpt-4",
  preamble: "You research information thoroughly and provide facts."
});

const analyzerAgent = new Agent({
  name: "Analyzer",
  model: "gpt-4",
  preamble: "You analyze information and provide insights."
});

// Create a sequential process
const process = new Process()
  .addStep({
    agent: researchAgent,
    prompt: (input) => `Research about ${input}`,
    outputKey: "research"
  })
  .addStep({
    agent: analyzerAgent,
    prompt: (context) => `Analyze this research: ${context.research}`,
    outputKey: "analysis"
  });

// Execute the process
async function main() {
  const result = await process.execute("Ethereum scaling solutions");
  console.log("Research:", result.research);
  console.log("Analysis:", result.analysis);
}

main();
```

> **See Also:** Learn more about coordinating multiple agents in [Multi-Agent Systems](Advanced/MultiAgentSystems.md) and see a complete implementation in [Multi-Agent Tutorial](Tutorials/MultiAgentSystem.md).

## Related Documentation

* [Tools](Features/Tools.md) - Learn more about extending agent capabilities with custom tools
* [Memory](Features/Memory.md) - Detailed guide on implementing different memory systems
* [Knowledge](Features/Knowledge.md) - Add external knowledge sources to your agents
* [Store](Features/Store.md) - Persist and retrieve data across sessions
* [Web3 Integration](Integrations/Web3Integration.md) - Advanced blockchain integration patterns
* [Telegram Bot Tutorial](Tutorials/TelegramBot.md) - Build a complete Telegram bot with Alith
* [DeFi Advisor Tutorial](Tutorials/DeFiAdvisor.md) - Create a DeFi portfolio advisor

## API Reference

```typescript
// Agent interface (TypeScript)
interface AgentConfig {
  name?: string;
  model: string | LLMConfig;
  preamble?: string;
  tools?: Tool[];
  memory?: MemoryConfig;
  timeout?: number;
  maxRetries?: number;
  streaming?: boolean;
  rateLimiter?: RateLimiter;
}

class Agent {
  constructor(config: AgentConfig);
  async prompt(text: string): Promise<string>;
  async promptStream(text: string, callback: (chunk: string) => void): Promise<string>;
  async reset(): Promise<void>;
}

// Tool interface
interface ToolSchema {
  type: string;
  properties?: Record<string, ToolParameter>;
  required?: string[];
}

interface ToolParameter {
  type: string;
  description?: string;
  properties?: Record<string, ToolParameter>;
  required?: string[];
}

interface Tool {
  name: string;
  description: string;
  schema: ToolSchema;
  function: (params: any) => Promise<any> | any;
}

// Memory interfaces
interface MemoryConfig {
  type: 'buffer' | 'summarizing' | 'persistent';
  capacity: number;
  tokenLimit?: number;
  summarizeAfter?: number;
  storage?: 'memory' | 'redis' | 'mongo';
  connectionString?: string;
}
```

---

*Last Updated: 2023-10-15*