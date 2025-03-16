# LLMs - Language Model Integration in Alith

## Overview

Language models are the core intelligence behind Alith agents, enabling them to understand natural language, generate human-like responses, reason through complex problems, and interact with tools. Alith provides a unified interface to work with various LLM providers, including OpenAI, Anthropic, and other API-compatible services.

> **🧠 Core Feature**: LLMs power every aspect of an Alith agent, from understanding user inputs to generating responses, reasoning about information, and using tools effectively.

## Key Concepts

* **Model Provider**: The company or service offering the language model (OpenAI, Anthropic, etc.)
* **Model Name**: The specific model variant to use (e.g., "gpt-4", "claude-3-5-sonnet")
* **Context Window**: Maximum token limit for a single interaction with the model
* **Temperature**: Controls randomness in model responses (higher values produce more creative outputs)
* **API Compatibility**: Support for models implementing OpenAI-compatible APIs
* **Streaming**: Real-time generation of model responses as tokens are produced

## Usage

Alith supports multiple LLM providers and offers consistent ways to integrate them into your applications.

### OpenAI Models (Rust)

```rust
use alith::{Agent, LLM};
 
#[tokio::main]
async fn main() -> Result<(), anyhow::Error> {
    // Set API key in environment: OPENAI_API_KEY=your-api-key
    
    // Create a model instance
    let model = LLM::from_model_name("gpt-4")?;
    
    // Create an agent using the model
    let agent = Agent::new("assistant", model, vec![])
        .preamble("You are a helpful assistant who provides clear, accurate information.");
 
    // Interact with the agent
    let response = agent.prompt("What are the key features of Rust programming language?").await?;
 
    println!("{}", response);
 
    Ok(())
}
```

### Anthropic Models (Rust)

```rust
use alith::{Agent, LLM};
 
#[tokio::main]
async fn main() -> Result<(), anyhow::Error> {
    // Set API key in environment: ANTHROPIC_API_KEY=your-api-key
    
    // Create a model instance
    let model = LLM::from_model_name("claude-3-5-sonnet")?;
    
    // Create an agent using the model
    let agent = Agent::new("research assistant", model, vec![])
        .preamble("You are a research assistant who helps with complex queries and analysis.");
 
    // Interact with the agent
    let response = agent.prompt("Explain the difference between transformers and RNNs in deep learning.").await?;
 
    println!("{}", response);
 
    Ok(())
}
```

### OpenAI API-Compatible Models (Rust)

Integrate with third-party models that implement OpenAI-compatible APIs:

```rust
use alith::{Agent, LLM};
 
#[tokio::main]
async fn main() -> Result<(), anyhow::Error> {
    // Create a model instance for a compatible API
    let model = LLM::openai_compatible_model(
        "your-api-key",         // Replace with your API key or read from env
        "api.deepseek.com",     // The API endpoint
        "deepseek-chat",        // The model name (or "deepseek-reasoner" for DeepSeek R1)
    )?;
    
    // Create an agent using the model
    let agent = Agent::new("code assistant", model, vec![])
        .preamble("You are a coding assistant who helps with programming problems.");
 
    // Interact with the agent
    let response = agent.prompt("How do I implement a binary search tree in Python?").await?;
 
    println!("{}", response);
 
    Ok(())
}
```

### Node.js LLM Implementation

```javascript
const { Agent, LLM } = require('alith');

// Create an agent with OpenAI model
async function createOpenAIAgent() {
  const openaiModel = new LLM({
    provider: 'openai',
    model: 'gpt-4o',
    apiKey: process.env.OPENAI_API_KEY,
    temperature: 0.7
  });

  const agent = new Agent({
    name: "TechSupport",
    model: openaiModel,
    preamble: "You are a technical support specialist who helps users solve computer problems."
  });

  const response = await agent.prompt(
    "My laptop keeps overheating and shutting down. What should I check?"
  );
  
  console.log(response);
}

// Create an agent with Anthropic model
async function createAnthropicAgent() {
  const claudeModel = new LLM({
    provider: 'anthropic',
    model: 'claude-3-opus-20240229',
    apiKey: process.env.ANTHROPIC_API_KEY,
    temperature: 0.5
  });

  const agent = new Agent({
    name: "WritingAssistant",
    model: claudeModel,
    preamble: "You are a writing assistant who helps improve text clarity and structure."
  });

  const response = await agent.prompt(
    "Please help me revise this paragraph: The impact of climate change affects many sectors. It causes problems. We should act now."
  );
  
  console.log(response);
}

createOpenAIAgent();
createAnthropicAgent();
```

### Python LLM Implementation

```python
from alith import Agent, LLM

# Create an OpenAI-powered agent
def create_openai_agent():
    # API key from environment variable OPENAI_API_KEY
    openai_model = LLM(
        provider="openai",
        model="gpt-4-turbo",
        temperature=0.7,
        max_tokens=2048
    )
    
    agent = Agent(
        name="DataAnalyst",
        model=openai_model,
        preamble="You are a data analysis expert who helps interpret datasets and statistics."
    )
    
    response = agent.prompt(
        "What are the best visualization techniques for showing correlation between variables?"
    )
    
    print(response)

# Create an Anthropic-powered agent
def create_anthropic_agent():
    # API key from environment variable ANTHROPIC_API_KEY
    claude_model = LLM(
        provider="anthropic",
        model="claude-3-sonnet-20240229",
        temperature=0.3
    )
    
    agent = Agent(
        name="HistoryTutor", 
        model=claude_model,
        preamble="You are a history tutor who explains historical events and their context."
    )
    
    response = agent.prompt(
        "Explain the causes and consequences of the Industrial Revolution."
    )
    
    print(response)

create_openai_agent()
create_anthropic_agent()
```

## Common Patterns

### Pattern 1: Streaming Responses

For improved user experience with real-time responses:

```javascript
const { Agent, LLM } = require('alith');

async function streamingExample() {
  const model = new LLM({
    provider: 'openai',
    model: 'gpt-4',
    streaming: true
  });

  const agent = new Agent({
    name: "StreamingAssistant",
    model: model,
    preamble: "You are an assistant that provides detailed step-by-step explanations."
  });

  // Stream the response
  await agent.promptStream(
    "Explain how photosynthesis works.",
    (chunk) => {
      // Process each chunk as it arrives
      process.stdout.write(chunk);
    }
  );
}

streamingExample();
```

### Pattern 2: Model Fallback

Implement fallback mechanisms for reliability:

```python
from alith import Agent, LLM
import time

def create_agent_with_fallback():
    # Primary model configuration
    primary_model = LLM(
        provider="openai",
        model="gpt-4",
        timeout=10  # Short timeout to quickly detect issues
    )
    
    # Fallback model configuration
    fallback_model = LLM(
        provider="anthropic",
        model="claude-3-haiku-20240307"
    )
    
    # Try primary model first
    try:
        agent = Agent(
            name="ReliableAssistant",
            model=primary_model,
            preamble="You are a helpful assistant."
        )
        
        response = agent.prompt("What is machine learning?")
        return response
    except Exception as e:
        print(f"Primary model failed: {e}")
        print("Falling back to alternative model...")
        
        # Wait a moment before retry
        time.sleep(1)
        
        # Use fallback model
        agent = Agent(
            name="ReliableAssistant",
            model=fallback_model,
            preamble="You are a helpful assistant."
        )
        
        response = agent.prompt("What is machine learning?")
        return response

result = create_agent_with_fallback()
print(result)
```

## Configuration Options

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `provider` | string | - | LLM provider (openai, anthropic, etc.) |
| `model` | string | - | Specific model name |
| `apiKey` | string | From env | API key for the provider |
| `temperature` | number | 0.7 | Controls randomness (0.0-1.0) |
| `maxTokens` | number | Varies | Maximum tokens in response |
| `topP` | number | 1.0 | Nucleus sampling parameter |
| `frequencyPenalty` | number | 0.0 | Penalizes frequent tokens |
| `presencePenalty` | number | 0.0 | Penalizes repeated tokens |
| `streaming` | boolean | false | Whether to stream responses |
| `timeout` | number | 60000 | API call timeout in milliseconds |

## Best Practices

* **Right-Size Models**: Choose models appropriate for your task complexity and budget
* **Effective Prompting**: Write clear, specific preambles for consistent agent behavior
* **Error Handling**: Implement robust handling for API outages and rate limits
* **Cost Management**: Monitor token usage to control expenses
* **Performance Optimization**: Use streaming for responsive UIs and cache common responses

> **💡 Tip:** For production applications, implement a model fallback system that can switch to alternative providers if your primary model service experiences issues.

## Common Issues and Solutions

### Issue 1: API Rate Limiting

Hitting rate limits during high usage periods.

**Solution**: Implement exponential backoff and request queuing:

```javascript
const { Agent, LLM } = require('alith');

class RateLimitHandler {
  constructor() {
    this.queue = [];
    this.processing = false;
    this.backoffTime = 1000; // Start with 1 second
  }
  
  async enqueue(prompt, agent) {
    return new Promise((resolve, reject) => {
      this.queue.push({ prompt, agent, resolve, reject });
      if (!this.processing) this.processQueue();
    });
  }
  
  async processQueue() {
    if (this.queue.length === 0) {
      this.processing = false;
      return;
    }
    
    this.processing = true;
    const { prompt, agent, resolve, reject } = this.queue.shift();
    
    try {
      const result = await agent.prompt(prompt);
      resolve(result);
      this.backoffTime = 1000; // Reset backoff on success
      // Process next item immediately
      this.processQueue();
    } catch (error) {
      if (error.message.includes('rate limit')) {
        console.log(`Rate limited. Backing off for ${this.backoffTime}ms`);
        // Put the request back at the front of the queue
        this.queue.unshift({ prompt, agent, resolve, reject });
        // Exponential backoff
        setTimeout(() => this.processQueue(), this.backoffTime);
        this.backoffTime *= 2; // Double backoff time for next failure
      } else {
        // For non-rate limit errors, reject the promise
        reject(error);
        this.processQueue(); // Continue with next request
      }
    }
  }
}

// Usage
const handler = new RateLimitHandler();
const model = new LLM({ provider: 'openai', model: 'gpt-4' });
const agent = new Agent({ name: 'assistant', model });

// Submit multiple requests that will be queued and processed with backoff
async function main() {
  const prompts = [
    "What is AI?",
    "Explain machine learning",
    "What are neural networks?",
    // More prompts...
  ];
  
  const results = await Promise.all(
    prompts.map(prompt => handler.enqueue(prompt, agent))
  );
  
  console.log(results);
}

main();
```

### Issue 2: Token Limits Exceeded

Attempting to process too much content for the model's context window.

**Solution**: Implement chunking and summarization strategies:

```python
from alith import Agent, LLM

def process_long_document(document, max_chunk_size=4000):
    # Split into chunks
    chunks = []
    current_chunk = ""
    
    for paragraph in document.split("\n\n"):
        if len(current_chunk) + len(paragraph) < max_chunk_size:
            current_chunk += paragraph + "\n\n"
        else:
            chunks.append(current_chunk)
            current_chunk = paragraph + "\n\n"
    
    if current_chunk:
        chunks.append(current_chunk)
    
    # Process each chunk
    model = LLM(provider="openai", model="gpt-4")
    agent = Agent(name="DocumentProcessor", model=model)
    
    # First get summaries of each chunk
    summaries = []
    for i, chunk in enumerate(chunks):
        prompt = f"Summarize this document chunk ({i+1}/{len(chunks)}):\n\n{chunk}"
        summary = agent.prompt(prompt)
        summaries.append(summary)
    
    # Then process the combined summaries
    combined = "\n\n".join(summaries)
    final_summary = agent.prompt(
        f"Create a comprehensive summary of this document based on these section summaries:\n\n{combined}"
    )
    
    return final_summary
```

## Advanced Usage

### Custom LLM Provider Integration

Integrate with custom or self-hosted LLM services:

```javascript
const { BaseLLM } = require('alith');
const axios = require('axios');

class CustomLLM extends BaseLLM {
  constructor(options) {
    super();
    this.endpoint = options.endpoint;
    this.apiKey = options.apiKey;
    this.model = options.model || 'default';
    this.temperature = options.temperature || 0.7;
  }
  
  async generate(prompt, options = {}) {
    try {
      const response = await axios.post(this.endpoint, {
        prompt: prompt,
        model: this.model,
        temperature: options.temperature || this.temperature,
        max_tokens: options.maxTokens || 1000
      }, {
        headers: {
          'Authorization': `Bearer ${this.apiKey}`,
          'Content-Type': 'application/json'
        }
      });
      
      return response.data.text || response.data.choices[0].text;
    } catch (error) {
      console.error('Error generating text:', error);
      throw new Error(`Text generation failed: ${error.message}`);
    }
  }
  
  async generateStream(prompt, callback, options = {}) {
    // Implement streaming functionality
    // This is more complex and depends on the API
  }
}

// Usage
const customModel = new CustomLLM({
  endpoint: 'https://api.customllm.com/generate',
  apiKey: process.env.CUSTOM_API_KEY,
  model: 'custom-large-v2'
});

const agent = new Agent({
  name: "CustomAssistant",
  model: customModel,
  preamble: "You are an assistant powered by a custom LLM."
});
```

## Related Documentation

* [Tools](Tools.md) - Learn how LLMs interact with tools
* [Memory](Memory.md) - How LLMs use context from previous interactions
* [Knowledge](Knowledge.md) - Enhancing LLMs with external knowledge
* [Embeddings](Embeddings.md) - Compare with embedding models
* [Integration Tutorial: OpenAI](../Integrations/OpenAIIntegration.md) - Detailed OpenAI setup guide

## API Reference

```typescript
// LLM interfaces
interface LLMConfig {
  provider: 'openai' | 'anthropic' | 'custom';
  model: string;
  apiKey?: string;
  temperature?: number;
  maxTokens?: number;
  topP?: number;
  frequencyPenalty?: number;
  presencePenalty?: number;
  streaming?: boolean;
  timeout?: number;
}

interface LLMResponse {
  text: string;
  modelName: string;
  tokenUsage: {
    prompt: number;
    completion: number;
    total: number;
  };
  finishReason: string;
}

interface LLMStreamOptions {
  onToken: (token: string) => void;
  onComplete?: (response: LLMResponse) => void;
  onError?: (error: Error) => void;
}
```

---

*Last Updated: 2025-03-16*
