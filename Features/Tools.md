# Tools - Extending Agent Capabilities

## Overview

Tools in Alith allow you to define custom functions that agents can use to perform specific tasks. They are the primary way to extend your agent's capabilities beyond conversation, enabling operations like calculations, API calls, blockchain interactions, database queries, and more.

> **🔧 Core Feature**: Tools transform your agents from simple conversational interfaces into powerful automation systems that can interact with external services and perform complex operations.

## Key Concepts

* **Tool**: A function or class that encapsulates logic that an agent can invoke
* **StructureTool**: A tool with structured inputs and outputs (strongly typed)
* **Tool Input**: The parameters a tool accepts, typically defined as a struct or schema
* **Tool Output**: The result a tool returns after execution
* **Tool Description**: Information that helps the agent understand when and how to use the tool

## Usage

Tools are defined differently depending on your programming language, but follow the same core principles.

### Rust

```rust
use alith::{Agent, StructureTool, Tool, ToolError, LLM};
use async_trait::async_trait;
use schemars::JsonSchema;
use serde::{Deserialize, Serialize};
 
#[derive(JsonSchema, Serialize, Deserialize)]
pub struct Input {
    pub x: usize,
    pub y: usize,
}
 
pub struct Adder;
#[async_trait]
impl StructureTool for Adder {
    type Input = Input;
    type Output = usize;
 
    fn name(&self) -> &str {
        "adder"
    }
 
    fn description(&self) -> &str {
        "Add x and y together"
    }
 
    async fn run_with_args(&self, input: Self::Input) -> Result<Self::Output, ToolError> {
        let result = input.x + input.y;
        Ok(result)
    }
}
 
pub struct Subtract;
#[async_trait]
impl StructureTool for Subtract {
    type Input = Input;
    type Output = usize;
 
    fn name(&self) -> &str {
        "subtract"
    }
 
    fn description(&self) -> &str {
        "Subtract y from x (i.e.: x - y)"
    }
 
    async fn run_with_args(&self, input: Self::Input) -> Result<Self::Output, ToolError> {
        let result = input.x - input.y;
        Ok(result)
    }
}
 
#[tokio::main]
async fn main() -> Result<(), anyhow::Error> {
    let tools: [Box<dyn Tool>; 2] = [Box::new(Adder), Box::new(Subtract)];
    let model = LLM::from_model_name("gpt-4")?;
    let agent = Agent::new("simple agent", model, tools)
        .preamble("You are a calculator here to help the user perform arithmetic operations. Use the tools provided to answer the user's question.");
    let response = agent.prompt("Calculate 10 - 3").await?;
 
    println!("{}", response);
 
    Ok(())
}

### Python

```python
from alith import Agent, Tool
from pydantic import BaseModel

class CalculatorInput(BaseModel):
    x: int
    y: int

def adder(input: CalculatorInput) -> int:
    """Add x and y together"""
    return input.x + input.y

# Create and use the tool
calculator_tool = Tool(
    name="adder",
    description="Add two numbers together",
    function=adder,
    input_schema=CalculatorInput
)

agent = Agent(
    name="calculator",
    model="gpt-4",
    tools=[calculator_tool]
)

response = agent.prompt("Calculate 5 + 7")
print(response)
```

### Node.js

```javascript
const { Agent, Tool } = require('alith');

// Simple tool definition
const calculatorTool = new Tool({
  name: 'calculator',
  description: 'Performs basic arithmetic operations',
  schema: {
    type: 'object',
    properties: {
      operation: { type: 'string', enum: ['add', 'subtract', 'multiply', 'divide'] },
      x: { type: 'number' },
      y: { type: 'number' }
    },
    required: ['operation', 'x', 'y']
  },
  function: async ({ operation, x, y }) => {
    switch (operation) {
      case 'add': return x + y;
      case 'subtract': return x - y;
      case 'multiply': return x * y;
      case 'divide': return x / y;
    }
  }
});

const agent = new Agent({
  name: 'calculator assistant',
  model: 'gpt-4',
  tools: [calculatorTool]
});

agent.prompt('Calculate 10 - 3').then(response => {
  console.log(response);
});
```

## Common Patterns

### Pattern 1: Web API Integration

Tools frequently integrate with external APIs to fetch real-time data.

```javascript
const { Tool } = require('alith');
const axios = require('axios');

const cryptoPriceTool = new Tool({
  name: 'cryptoPrice',
  description: 'Get the current price of a cryptocurrency in USD',
  schema: {
    type: 'object',
    properties: {
      symbol: { type: 'string', description: 'Cryptocurrency symbol (e.g., BTC, ETH)' }
    },
    required: ['symbol']
  },
  function: async ({ symbol }) => {
    try {
      const response = await axios.get(
        `https://api.coingecko.com/api/v3/simple/price?ids=${symbol.toLowerCase()}&vs_currencies=usd`
      );
      return response.data[symbol.toLowerCase()]?.usd || 'Price not found';
    } catch (error) {
      return `Error fetching price: ${error.message}`;
    }
  }
});
```

### Pattern 2: Builtin Tools

Alith provides several built-in tools for common operations like web searches.

```rust
use alith::{Agent, SearchTool, Tool, LLM};
 
#[tokio::main]
async fn main() -> Result<(), anyhow::Error> {
    let tools: [Box<dyn Tool>; 1] = [Box::new(SearchTool::default())];
    let model = LLM::from_model_name("gpt-4")?;
    let agent = Agent::new("simple agent", model, tools)
        .preamble("You are a searcher. When I ask questions about Web3, you can search from the Internet and answer them. When you encounter other questions, you can directly answer them.");
    let response = agent.prompt("What's BitCoin?").await?;
 
    println!("{}", response);
 
    Ok(())
}
```

## Configuration Options

| Option | Type | Description |
|--------|------|-------------|
| `name` | string | Unique identifier for the tool |
| `description` | string | Detailed description of what the tool does and when to use it |
| `schema` | object | JSON Schema defining the input parameters |
| `function` | function | Implementation that performs the tool's operation |
| `timeout` | number | Optional timeout in milliseconds |
| `streaming` | boolean | Whether the tool supports streaming responses |
| `cache` | object | Optional caching configuration for tool results |

## Best Practices

* **Clear Descriptions**: Write clear, specific tool descriptions so the agent knows when to use them
* **Error Handling**: Always implement proper error handling in your tools
* **Input Validation**: Validate inputs before processing to prevent runtime errors
* **Timeouts**: Implement timeouts for API calls to prevent hanging
* **Stateless Design**: Design tools to be stateless when possible for better reliability

> **⚠️ Warning:** Tools with unclear descriptions or poor error handling can lead to unpredictable agent behavior. Always provide comprehensive descriptions and robust error handling.

## Common Issues and Solutions

### Issue 1: Agent Not Using Tools

The agent sometimes ignores available tools and tries to answer directly.

**Solution**: Improve the tool description and add explicit instructions in the agent's preamble to use tools for specific types of queries.

### Issue 2: Tool Execution Errors

Tools fail during execution due to unexpected inputs or API failures.

**Solution**: Implement comprehensive error handling and return informative error messages that help the agent adjust its approach.

```javascript
function: async (params) => {
  try {
    // Tool implementation
    return result;
  } catch (error) {
    // Detailed error message
    return `Could not complete the operation because: ${error.message}. 
            Try with different parameters or a different approach.`;
  }
}
```

## Advanced Usage

### Chaining Multiple Tools

Tools can be designed to work together in sequence for complex operations:

```javascript
const { Agent, Tool } = require('alith');

// First tool fetches data
const fetchDataTool = new Tool({
  name: 'fetchData',
  description: 'Fetch raw data from source',
  // Tool implementation
});

// Second tool processes data
const processDataTool = new Tool({
  name: 'processData',
  description: 'Process raw data into insights',
  // Tool implementation
});

const agent = new Agent({
  name: 'data analyst',
  model: 'gpt-4',
  tools: [fetchDataTool, processDataTool],
  preamble: `You are a data analyst assistant. For complex analysis tasks, 
             first use fetchData to get the raw data, then use processData 
             to generate insights from that data.`
});
```

### Creating Custom Tool Classes

For more complex tools, you can create custom tool classes that encapsulate related functionality:

```typescript
class DatabaseTool extends BaseTool {
  private connection: DatabaseConnection;
  
  constructor(connectionString: string) {
    super();
    this.connection = new DatabaseConnection(connectionString);
  }
  
  async query(sql: string): Promise<any[]> {
    return this.connection.execute(sql);
  }
  
  get toolDefinitions() {
    return [
      {
        name: 'queryDatabase',
        description: 'Execute a SQL query against the database',
        schema: {
          type: 'object',
          properties: {
            query: { type: 'string', description: 'SQL query to execute' }
          },
          required: ['query']
        },
        function: async ({ query }) => this.query(query)
      },
      {
        name: 'listTables',
        description: 'List all tables in the database',
        schema: { type: 'object', properties: {} },
        function: async () => this.query('SHOW TABLES')
      }
    ];
  }
}

// Usage
const dbTool = new DatabaseTool('connection_string');
const agent = new Agent({
  name: 'database assistant',
  model: 'gpt-4',
  tools: dbTool.toolDefinitions
});
```

## Related Documentation

* [Memory](Memory.md) - Learn how to maintain state between tool invocations
* [LLMs](LLMs.md) - Configure the language models that interpret and use tools
* [Web3 Integration](../Integrations/Web3Integration.md) - Create blockchain-specific tools
* [Tutorial: Building a Telegram Bot](../Tutorials/TelegramBot.md) - See tools in action in a real-world application
* [Tutorial: Building a Blockchain Data Analyzer](../Tutorials/BlockchainAnalyzer.md) - Example of specialized Web3 tools

## API Reference

```typescript
// TypeScript interface for Tool
interface ToolDefinition<T, R> {
  name: string;
  description: string;
  schema: JsonSchema;
  function: (input: T) => Promise<R>;
  timeout?: number; // Optional timeout in milliseconds
}

class Tool<T, R> {
  constructor(definition: ToolDefinition<T, R>);
  async execute(input: T): Promise<R>;
}

// Tool error interface
interface ToolError {
  message: string;
  code?: string;
  details?: any;
}
```

---

*Last Updated: 2025-03-16*