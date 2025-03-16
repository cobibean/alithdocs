# Memory - Conversation History for Alith Agents

## Overview

The Alith Memory system allows agents to retain and recall information across multiple interactions. This is essential for building conversational agents that can maintain context, remember user preferences, and provide coherent multi-turn conversations. With memory, your agents become more personable and effective by building on previous exchanges.

> **Note:** Memory systems help create more natural interactions but consume tokens from your AI model's context window. Choose appropriate memory types and capacities based on your use case.

## Key Concepts

* **Memory System**: A component that stores and retrieves conversation history
* **Memory Types**: Different implementations optimized for various use cases
* **Memory Capacity**: The number of interactions or tokens to retain
* **Context Management**: How memory content is formatted and sent to the AI model
* **Persistence**: Whether memory is stored only in-memory or persisted to external storage

## Usage

Alith provides several memory implementations that can be easily added to your agents.

### Window Buffer Memory (Rust)

The Window Buffer Memory system keeps a fixed number of recent interactions:

```rust
use alith::{Agent, WindowBufferMemory, LLM};
 
#[tokio::main]
async fn main() -> Result<(), anyhow::Error> {
    let model = LLM::from_model_name("gpt-4")?;
    let mut agent = Agent::new("simple agent", model, vec![])
        .preamble("You are a helpful assistant that remembers our conversation.")
        .memory(WindowBufferMemory::new(10)); // Remember last 10 interactions
    
    // First interaction
    let response = agent.prompt("My name is Alice.").await?;
    println!("{}", response);
    
    // Second interaction - agent will remember the name
    let response = agent.prompt("What's my name?").await?;
    println!("{}", response);
 
    Ok(())
}
```

### RLU Cache Memory (Rust)

RLU (Recently Least Used) Cache Memory optimizes for frequently accessed information:

```rust
use alith::{Agent, RLUCacheMemory, LLM};
 
#[tokio::main]
async fn main() -> Result<(), anyhow::Error> {
    let model = LLM::from_model_name("gpt-4")?;
    let mut agent = Agent::new("simple agent", model, vec![])
        .preamble("You are a helpful assistant that remembers our conversation.")
        .memory(RLUCacheMemory::new(10)); // Cache size of 10
    
    let response = agent.prompt("Remember that my favorite color is blue.").await?;
    println!("{}", response);
    
    // Later interactions
    let response = agent.prompt("What's my favorite color?").await?;
    println!("{}", response);
 
    Ok(())
}
```

### Node.js Memory Implementation

```javascript
const { Agent } = require('alith');

// Create an agent with buffer memory
const agent = new Agent({
  name: "MemoryAgent",
  model: "gpt-4",
  preamble: "You are a helpful assistant with memory capabilities.",
  memory: {
    type: "buffer",
    capacity: 5 // Remember last 5 interactions
  }
});

// Example conversation
async function demonstrateMemory() {
  console.log(await agent.prompt("My name is Bob."));
  console.log(await agent.prompt("I live in New York."));
  console.log(await agent.prompt("What's my name and where do I live?"));
}

demonstrateMemory();
```

### Python Memory Implementation

```python
from alith import Agent, BufferMemory

# Create an agent with memory
agent = Agent(
    name="MemoryAgent",
    model="gpt-4",
    preamble="You are a helpful assistant with memory capabilities.",
    memory=BufferMemory(capacity=5)  # Remember last 5 interactions
)

# Example conversation
print(agent.prompt("My name is Charlie."))
print(agent.prompt("I'm a software engineer."))
print(agent.prompt("What's my name and what do I do?"))
```

## Common Patterns

### Pattern 1: Summarizing Memory

For longer conversations, summarization can help manage token usage:

```javascript
const agent = new Agent({
  name: "SummarizingAgent",
  model: "gpt-4",
  preamble: "You are a helpful assistant that remembers important details.",
  memory: {
    type: "summarizing",
    capacity: 10,
    summarizeAfter: 8  // Summarize when we reach 8 messages
  }
});
```

### Pattern 2: Persistent Memory

For applications that need to retain memory across sessions:

```javascript
const { Agent, PersistentMemory } = require('alith');

const agent = new Agent({
  name: "PersistentAgent",
  model: "gpt-4",
  preamble: "You are a helpful assistant that remembers users across sessions.",
  memory: new PersistentMemory({
    storage: "redis",  // Store in Redis
    connectionString: process.env.REDIS_URL,
    prefix: "user_123_"  // User-specific prefix
  })
});
```

## Configuration Options

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `type` | string | `'buffer'` | Memory type (buffer, summarizing, persistent) |
| `capacity` | number | `10` | Number of interactions to store |
| `tokenLimit` | number | `4000` | Maximum tokens to send to model |
| `summarizeAfter` | number | `null` | When to trigger summarization |
| `storage` | string | `'memory'` | Where to store memory (memory, redis, etc.) |

## Best Practices

* **Right-Size Capacity**: Keep memory capacity appropriate for your model's context window
* **Consider Token Usage**: Memory consumes tokens, which affects both cost and model performance
* **Use Summarization**: For longer conversations, use summarization to compress history
* **User Identification**: In multi-user applications, ensure memories are associated with the correct user
* **Privacy Considerations**: Be transparent about what information is being stored in memory

## Common Issues and Solutions

### Issue 1: Token Limits Exceeded

When memory grows too large for your model's context window.

**Solution**: Use summarization or adjust memory capacity:

```javascript
const agent = new Agent({
  // ...configuration
  memory: {
    type: "summarizing",
    capacity: 15,
    tokenLimit: 3000,  // Ensure we stay under token limits
    summarizeAfter: 10
  }
});
```

### Issue 2: Memory Persistence

Memory is lost when the application restarts.

**Solution**: Implement persistent storage:

```python
from alith import Agent, RedisMemory

agent = Agent(
    # ...configuration
    memory=RedisMemory(
        connection_string="redis://localhost:6379",
        user_id="user_456",
        ttl=86400  # 24 hour expiration
    )
)
```

## Advanced Usage

### Custom Memory Implementation

You can create custom memory systems for specialized needs:

```javascript
const { Agent, BaseMemory } = require('alith');

class PrioritizedMemory extends BaseMemory {
  constructor(options) {
    super();
    this.capacity = options.capacity || 10;
    this.memories = [];
    this.priorities = {};
  }
  
  async add(interaction) {
    // Assign priority based on content analysis
    const priority = await this.analyzePriority(interaction);
    
    // Store with priority
    this.memories.push(interaction);
    this.priorities[interaction.id] = priority;
    
    // Prune if over capacity
    if (this.memories.length > this.capacity) {
      this.pruneLowestPriority();
    }
  }
  
  async get() {
    // Sort by priority before returning
    return this.memories.sort((a, b) => 
      this.priorities[b.id] - this.priorities[a.id]
    );
  }
  
  // Helper methods
  async analyzePriority(interaction) { /* implementation */ }
  pruneLowestPriority() { /* implementation */ }
}

// Use custom memory
const agent = new Agent({
  // ...configuration
  memory: new PrioritizedMemory({ capacity: 15 })
});
```

## Related Documentation

* [Memory Integration Tutorial](../Tutorials/MemoryTutorial.md) - Practical example of memory integration
* [LLMs](LLMs.md) - How memory interacts with different language models
* [Store](Store.md) - Persistent storage options for memory systems
* [Tutorials/TelegramBot.md](../Tutorials/TelegramBot.md#adding-memory) - See memory in action in a Telegram bot

## API Reference

```typescript
// Memory interfaces
interface MemoryConfig {
  type: 'buffer' | 'summarizing' | 'persistent';
  capacity: number;
  tokenLimit?: number;
  summarizeAfter?: number;
  storage?: 'memory' | 'redis' | 'mongo';
  connectionString?: string;
}

interface MemorySystem {
  add(interaction: Interaction): Promise<void>;
  get(): Promise<Interaction[]>;
  clear(): Promise<void>;
}

// Interaction object
interface Interaction {
  id: string;
  role: 'user' | 'assistant' | 'system';
  content: string;
  timestamp: number;
  metadata?: Record<string, any>;
}
```

---

*Last Updated: 2025-03-16*