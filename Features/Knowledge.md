# Knowledge - Enhancing Agents with External Information

## Overview

The Alith Knowledge system allows agents to access structured or unstructured data for more informed and context-aware responses. You can integrate databases, document stores, or custom knowledge sources to enhance your agents' capabilities, enabling them to reference specific information beyond their training data.

> **🧠 Core Feature**: Knowledge bases transform your agents from general-purpose assistants into domain experts that can provide accurate, up-to-date information from your custom data sources.

## Key Concepts

* **Knowledge Source**: Any external data source that can be accessed by the agent
* **Retrieval**: The process of finding relevant information from a knowledge source
* **Embedding**: Vector representation of text used for semantic search
* **Context Augmentation**: Adding retrieved knowledge to the agent's context
* **Knowledge Types**: Different formats of knowledge (text, PDFs, HTML, databases, etc.)

## Usage

Alith provides several knowledge source implementations that can be easily added to your agents.

### String Source Knowledge (Rust)

Use string knowledge to directly provide text content to your agent:

```rust
use alith::{Agent, LLM, Knowledge, StringKnowledge};
use std::sync::Arc;
 
#[tokio::main]
async fn main() -> Result<(), anyhow::Error> {
    let knowledges: Vec<Box<dyn Knowledge>> = vec![
        Box::new(StringKnowledge::new("The product warranty covers manufacturing defects for 2 years from purchase date.")),
    ];
    let model = LLM::from_model_name("gpt-4")?;
    let mut agent = Agent::new("support agent", model, vec![])
        .preamble("You are a customer support agent who provides accurate warranty information.");
    agent.knowledges = Arc::new(knowledges);
    let response = agent.prompt("How long is the warranty period?").await?;
 
    println!("{}", response);
 
    Ok(())
}
```

### Text File Knowledge (Rust)

Load knowledge from text files for larger content:

```rust
use alith::{Agent, LLM, Knowledge, TextFileKnowledge};
use std::sync::Arc;
 
#[tokio::main]
async fn main() -> Result<(), anyhow::Error> {
    let knowledges: Vec<Box<dyn Knowledge>> = vec![
        Box::new(TextFileKnowledge::new("data/product_manual.txt")),
    ];
    let model = LLM::from_model_name("gpt-4")?;
    let mut agent = Agent::new("product expert", model, vec![])
        .preamble("You are a product expert who helps users understand product features and troubleshooting steps.");
    agent.knowledges = Arc::new(knowledges);
    let response = agent.prompt("How do I reset the device to factory settings?").await?;
 
    println!("{}", response);
 
    Ok(())
}
```

### PDF Source Knowledge (Rust)

Extract knowledge from PDF documents:

```rust
use alith::{Agent, LLM, Knowledge, PdfFileKnowledge};
use std::sync::Arc;
 
#[tokio::main]
async fn main() -> Result<(), anyhow::Error> {
    let knowledges: Vec<Box<dyn Knowledge>> = vec![
        Box::new(PdfFileKnowledge::new("data/research_paper.pdf")),
    ];
    let model = LLM::from_model_name("gpt-4")?;
    let mut agent = Agent::new("research assistant", model, vec![])
        .preamble("You are a research assistant who helps summarize and explain scientific papers.");
    agent.knowledges = Arc::new(knowledges);
    let response = agent.prompt("What are the key findings of this paper?").await?;
 
    println!("{}", response);
 
    Ok(())
}
```

### HTML Source Knowledge (Rust)

Parse and extract knowledge from web content:

```rust
use alith::{Agent, LLM, HtmlKnowledge, Knowledge};
use std::io::Cursor;
use std::sync::Arc;
use url::Url;
 
#[tokio::main]
async fn main() -> Result<(), anyhow::Error> {
    let url = "https://en.m.wikivoyage.org/wiki/Seoul";
    let html = reqwest::get(url).await.unwrap().text().await.unwrap();
 
    let knowledges: Vec<Box<dyn Knowledge>> = vec![
        Box::new(HtmlKnowledge::new(
            Cursor::new(html),
            Url::parse(url).unwrap(),
            false, // Don't include images
        )),
    ];
    let model = LLM::from_model_name("gpt-4")?;
    let mut agent = Agent::new("travel guide", model, vec![])
        .preamble("You are a travel guide who provides detailed information about destinations.");
    agent.knowledges = Arc::new(knowledges);
    let response = agent.prompt("What are the must-see attractions in Seoul?").await?;
 
    println!("{}", response);
 
    Ok(())
}
```

### Node.js Knowledge Implementation

```javascript
const { Agent, Knowledge } = require('alith');

// Create an agent with text knowledge
const agent = new Agent({
  name: "KnowledgeAgent",
  model: "gpt-4",
  preamble: "You are a financial advisor who provides guidance based on up-to-date market information.",
  knowledge: [
    {
      type: "text",
      source: "Market trends show a 5% increase in tech stocks over the last quarter, with renewable energy stocks showing the highest growth at 12%.",
    },
    {
      type: "file",
      source: "./data/financial_report_2023.txt"
    }
  ]
});

// Example usage
async function demonstrateKnowledge() {
  console.log(await agent.prompt("What sectors are performing well in the market?"));
}

demonstrateKnowledge();
```

### Python Knowledge Implementation

```python
from alith import Agent, TextKnowledge, FileKnowledge, URLKnowledge

# Create various knowledge sources
text_knowledge = TextKnowledge(
    "The human genome contains approximately 3 billion base pairs."
)

file_knowledge = FileKnowledge(
    file_path="./data/genomics_research.txt"
)

url_knowledge = URLKnowledge(
    url="https://example.com/latest_research"
)

# Create an agent with multiple knowledge sources
agent = Agent(
    name="GenomicsExpert",
    model="gpt-4",
    preamble="You are a genomics expert who helps explain complex genetic concepts in simple terms.",
    knowledge=[text_knowledge, file_knowledge, url_knowledge]
)

# Example usage
response = agent.prompt("How many base pairs are in the human genome?")
print(response)
```

## Common Patterns

### Pattern 1: Combining Multiple Knowledge Sources

For comprehensive coverage, combine different knowledge sources:

```javascript
const { Agent, Knowledge } = require('alith');

const agent = new Agent({
  name: "ComprehensiveExpert",
  model: "gpt-4",
  preamble: "You are an expert who provides accurate information from multiple sources.",
  knowledge: [
    { type: "text", source: "Key fact: The product launched in June 2023." },
    { type: "file", source: "./data/product_specs.txt" },
    { type: "pdf", source: "./data/technical_documentation.pdf" },
    { type: "url", source: "https://example.com/latest_updates" }
  ]
});
```

### Pattern 2: Vector Store Knowledge

For larger knowledge bases, use vector stores for efficient retrieval:

```javascript
const { Agent, VectorStoreKnowledge } = require('alith');

// Initialize a vector store
const vectorStore = new VectorStoreKnowledge({
  documents: [
    "Document 1 with important information...",
    "Document 2 containing product specifications...",
    // Many more documents...
  ],
  embeddingModel: "text-embedding-ada-002",
  chunkSize: 1000,
  chunkOverlap: 200
});

// Create agent with vector store knowledge
const agent = new Agent({
  name: "KnowledgeAgent",
  model: "gpt-4",
  preamble: "You are an assistant with access to a large knowledge base.",
  knowledge: [vectorStore]
});
```

## Configuration Options

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `type` | string | - | Knowledge type (text, file, pdf, html, url, vector) |
| `source` | string | - | Content or path to the knowledge source |
| `chunkSize` | number | 1000 | Size of text chunks for processing large documents |
| `chunkOverlap` | number | 200 | Overlap between chunks to maintain context |
| `embeddingModel` | string | "text-embedding-ada-002" | Model used for creating embeddings |
| `maxTokens` | number | 3000 | Maximum tokens to include in context |
| `topK` | number | 5 | Number of most relevant chunks to include |

## Best Practices

* **Relevant Content**: Include only information relevant to your agent's purpose
* **Chunk Appropriately**: For large documents, ensure proper chunking for effective retrieval
* **Update Regularly**: Keep knowledge sources updated for accurate information
* **Combine with Tools**: Use knowledge alongside tools for powerful agent capabilities
* **Consider Context Limits**: Be mindful of token limits when adding knowledge to context

> **💡 Tip:** For large knowledge bases, implement a retrieval system that only adds the most relevant information to the context, rather than the entire knowledge base.

## Common Issues and Solutions

### Issue 1: Knowledge Not Being Used

Agent doesn't seem to reference the provided knowledge.

**Solution**: Make sure your agent's preamble explicitly instructs it to use the knowledge sources, and ensure the knowledge is relevant to the queries being asked.

### Issue 2: Context Window Exceeded

Too much knowledge being added to the context causes token limit errors.

**Solution**: Implement chunking and semantic search to only include the most relevant pieces of knowledge:

```javascript
const agent = new Agent({
  // ...configuration
  knowledge: {
    type: "vector",
    sources: [...documentSources],
    maxTokens: 2000,  // Limit tokens used for knowledge
    topK: 3  // Only include top 3 most relevant chunks
  }
});
```

## Advanced Usage

### Custom Knowledge Source Implementation

Create specialized knowledge sources for unique data:

```javascript
const { BaseKnowledge } = require('alith');

class APIKnowledge extends BaseKnowledge {
  constructor(apiEndpoint, apiKey) {
    super();
    this.apiEndpoint = apiEndpoint;
    this.apiKey = apiKey;
  }
  
  async retrieve(query) {
    // Call API to get relevant information
    const response = await fetch(`${this.apiEndpoint}?query=${encodeURIComponent(query)}`, {
      headers: { 'Authorization': `Bearer ${this.apiKey}` }
    });
    
    const data = await response.json();
    return data.results.map(item => item.content).join('\n\n');
  }
}

// Use custom knowledge source
const productKnowledge = new APIKnowledge(
  'https://api.company.com/product-data',
  process.env.API_KEY
);

const agent = new Agent({
  // ...configuration
  knowledge: [productKnowledge]
});
```

## Related Documentation

* [Memory](Memory.md) - How knowledge differs from and complements memory
* [Embeddings](Embeddings.md) - Learn about the embedding models used for knowledge retrieval
* [Store](Store.md) - Persistent storage options for knowledge bases
* [Tutorial: Building a RAG-based Telegram Bot](../Tutorials/TelegramBotWithRAG.md) - See knowledge in action in a practical application

## API Reference

```typescript
// Knowledge interfaces
interface KnowledgeConfig {
  type: 'text' | 'file' | 'pdf' | 'html' | 'url' | 'vector';
  source: string;
  chunkSize?: number;
  chunkOverlap?: number;
  embeddingModel?: string;
  maxTokens?: number;
  topK?: number;
}

interface KnowledgeSystem {
  retrieve(query: string): Promise<string>;
  getType(): string;
  getSource(): string;
}

// Vector store configuration
interface VectorStoreConfig {
  documents: string[];
  embeddingModel: string;
  chunkSize: number;
  chunkOverlap: number;
  similarity?: 'cosine' | 'euclidean' | 'dot';
}
```

---

*Last Updated: 2025-03-16*
