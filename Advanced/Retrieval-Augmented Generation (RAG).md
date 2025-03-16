# Retrieval-Augmented Generation (RAG)

## Overview

Retrieval-Augmented Generation (RAG) is a powerful technique that enhances AI agents by combining retrieval of relevant information from a knowledge base with text generation. This approach allows Alith agents to provide more accurate, up-to-date, and context-aware responses by leveraging external data sources.

> **🔍 Advanced Feature**: RAG significantly improves agent responses by grounding them in factual information from your custom knowledge sources, reducing hallucinations and improving accuracy.

## Key Concepts

* **Knowledge Retrieval**: Finding relevant information from a corpus of documents or data
* **Vector Embeddings**: Numerical representations of text used for semantic similarity search
* **Context Augmentation**: Adding retrieved knowledge to the agent's prompt context
* **Relevance Ranking**: Sorting retrieved information by relevance to the query
* **Knowledge Storage**: Systems for storing and indexing embeddings, including in-memory and vector databases

## When to Use RAG

RAG is particularly valuable when:

* Your agent needs access to specialized knowledge not in the model's training data
* You want to provide up-to-date information that emerged after the model's training cutoff
* You need the agent to reference specific documentation, policies, or proprietary information
* You want to reduce "hallucinations" by grounding responses in factual data

## Implementation Approaches

### In-Memory RAG (Rust)

For smaller knowledge bases or testing, you can use in-memory storage:

```rust
use alith::{Agent, EmbeddingsBuilder, InMemoryStorage, LLM};
 
#[tokio::main]
async fn main() -> Result<(), anyhow::Error> {
    // Initialize the language model
    let model = LLM::from_model_name("gpt-4")?;
    let embeddings_model = model.embeddings_model("text-embedding-3-small");
    
    // Create embeddings for your documents
    let data = EmbeddingsBuilder::new(embeddings_model.clone())
        .documents(vec![
            "Glarb-glarb is a fictional term used in science fiction to describe a type of alien communication.",
            "In the Zorblaxian language, glarb-glarb means 'greetings friend'.",
            "The origin of glarb-glarb dates back to the popular 1960s TV show 'Space Explorers'."
        ])
        .unwrap()
        .build()
        .await?;
    
    // Store embeddings in memory
    let storage = InMemoryStorage::from_multiple_documents(embeddings_model, data);
 
    // Create agent with the knowledge base
    let agent = Agent::new("dictionary assistant", model, vec![])
        .preamble(
            r#"You are a dictionary assistant here to assist the user in understanding word meanings.
You will find additional non-standard word definitions below that could be useful."#,
        )
        .store_index(1, storage);
    
    // Query the agent
    let response = agent.prompt("What does \"glarb-glarb\" mean?").await?;
 
    println!("{}", response);
 
    Ok(())
}
```

### Vector Database RAG (Rust)

For production applications with larger knowledge bases, use a vector database like Qdrant:

```rust
use alith::store::qdrant::{
    CreateCollectionBuilder, Distance, QdrantClient, QdrantStorage, VectorParamsBuilder,
    DEFAULT_COLLECTION_NAME,
};
use alith::{Agent, EmbeddingsBuilder, LLM};
 
#[tokio::main]
async fn main() -> Result<(), anyhow::Error> {
    // Initialize the language model
    let model = LLM::from_model_name("gpt-4")?;
    let embeddings_model = model.embeddings_model("text-embedding-3-small");
    
    // Create embeddings for your documents
    let documents = vec![
        "The Ethereum blockchain introduced smart contracts in 2015.", 
        "Smart contracts are self-executing programs with the terms directly written into code.",
        "Solidity is the most popular programming language for Ethereum smart contracts."
    ];
    
    let data = EmbeddingsBuilder::new(embeddings_model.clone())
        .documents(documents)
        .unwrap()
        .build()
        .await?;
 
    // Connect to Qdrant vector database
    let client = QdrantClient::from_url("http://localhost:6334").build()?;
 
    // Create collection if it doesn't exist
    if !client.collection_exists(DEFAULT_COLLECTION_NAME).await? {
        client
            .create_collection(
                CreateCollectionBuilder::new(DEFAULT_COLLECTION_NAME)
                    .vectors_config(VectorParamsBuilder::new(1536, Distance::Cosine)),
            )
            .await?;
    }
 
    // Store embeddings in Qdrant
    let storage = QdrantStorage::from_multiple_documents(client, embeddings_model, data).await?;
 
    // Create agent with the knowledge base
    let agent = Agent::new("blockchain expert", model, vec![])
        .preamble(
            r#"You are a blockchain expert assistant who provides accurate information about
blockchain technology, smart contracts, and related topics.
You will find additional technical information below that could be useful."#,
        )
        .store_index(1, storage);
    
    // Query the agent
    let response = agent.prompt("What are smart contracts and what language are they written in?").await?;
 
    println!("{}", response);
 
    Ok(())
}
```

### Node.js Implementation

```javascript
const { Agent, Embeddings, VectorStore } = require('alith');

async function createRagAgent() {
  // Initialize embeddings model
  const embeddings = new Embeddings({
    model: "text-embedding-ada-002"
  });

  // Create a vector store with documents
  const documents = [
    "The JavaScript language was created in 1995 by Brendan Eich.",
    "Node.js is a JavaScript runtime built on Chrome's V8 JavaScript engine.",
    "React is a JavaScript library for building user interfaces.",
    "TypeScript is a superset of JavaScript that adds static typing."
  ];

  // Generate embeddings and create vector store
  const vectorStore = await VectorStore.fromDocuments(documents, embeddings);
  
  // Create the RAG-enabled agent
  const agent = new Agent({
    name: "JavaScriptExpert",
    model: "gpt-4",
    preamble: `You are a JavaScript expert who provides accurate information about 
    JavaScript and related technologies. You'll be provided with relevant context
    from your knowledge base to help answer questions accurately.`,
    knowledge: vectorStore
  });
  
  // Query the agent
  const response = await agent.prompt("When was JavaScript created and who created it?");
  console.log(response);
}

createRagAgent();
```

### Python Implementation

```python
from alith import Agent, Embeddings, VectorStore
import os

async def create_rag_agent():
    # Initialize embeddings model
    embeddings = Embeddings(model="text-embedding-3-small")
    
    # Create documents for knowledge base
    documents = [
        "Python was created by Guido van Rossum and released in 1991.",
        "Python 3.0 was released in 2008 with many improvements over Python 2.x.",
        "The Zen of Python emphasizes readability and simplicity.",
        "Python is widely used in data science, machine learning, and web development."
    ]
    
    # Create vector store from documents
    vector_store = VectorStore.from_documents(documents, embeddings)
    
    # Create the RAG-enabled agent
    agent = Agent(
        name="PythonExpert",
        model="gpt-4",
        preamble="""You are a Python expert who provides accurate information about 
        Python programming. You'll be provided with relevant context from your 
        knowledge base to help answer questions accurately.""",
        knowledge=vector_store
    )
    
    # Query the agent
    response = await agent.prompt("Who created Python and when was it released?")
    print(response)

# Run the async function
import asyncio
asyncio.run(create_rag_agent())
```

## Configuration Options

| Option | Type | Description |
|--------|------|-------------|
| `maxChunks` | number | Maximum number of chunks to retrieve from the knowledge base |
| `similarityThreshold` | number | Minimum similarity score for retrieved chunks (0-1) |
| `includeSources` | boolean | Whether to include source references in the agent's response |
| `embeddingModel` | string | Model to use for generating embeddings |
| `chunkSize` | number | Size of text chunks when processing documents |
| `chunkOverlap` | number | Overlap between adjacent chunks to maintain context |

## Advanced RAG Patterns

### Pattern 1: Hybrid Search

Combine semantic search with keyword-based search for improved retrieval:

```javascript
const { HybridSearch } = require('alith');

const hybridSearch = new HybridSearch({
  vectorStore: vectorStore,
  keywordWeight: 0.3,  // 30% keyword, 70% semantic
  indexedFields: ['title', 'content']
});

const agent = new Agent({
  // ...agent configuration
  knowledge: hybridSearch
});
```

### Pattern 2: Multi-Query RAG

Generate multiple search queries from the user's input to improve retrieval:

```javascript
const { MultiQueryRetriever } = require('alith');

async function createMultiQueryAgent() {
  const retriever = new MultiQueryRetriever({
    vectorStore: vectorStore,
    queryCount: 3,  // Generate 3 different queries
    queryModel: "gpt-3.5-turbo"  // Use faster model for query generation
  });
  
  const agent = new Agent({
    // ...agent configuration
    knowledge: retriever
  });
  
  return agent;
}
```

## Best Practices

* **Pre-process Documents**: Clean and structure documents before embedding
* **Chunk Appropriately**: Adjust chunk size based on your content type
* **Test Retrieval Quality**: Evaluate retrieval quality with representative queries
* **Iterative Refinement**: Monitor and improve your knowledge base over time
* **Consider Storage Options**: Choose appropriate storage based on scale and performance needs

> **⚠️ Warning:** Large knowledge bases can significantly increase token usage. Implement filtering and relevance ranking to only include the most pertinent information in your agent's context.

## Common Issues and Solutions

### Issue 1: Irrelevant Information Retrieval

Retrieved information doesn't match the query intent.

**Solution**: Improve embedding quality and implement filtering:

```javascript
// Add metadata filtering to retrieval
const results = await vectorStore.similaritySearch(query, {
  k: 5,  // Number of results
  filter: { category: "technical", language: "english" }
});
```

### Issue 2: Context Window Limits

Too much retrieved content exceeds model context limits.

**Solution**: Implement summarization or prioritization:

```javascript
function prioritizeResults(results, query) {
  // Sort by relevance score
  const sorted = results.sort((a, b) => b.score - a.score);
  
  // Take top results until approaching token limit
  let tokenCount = 0;
  const selectedResults = [];
  
  for (const result of sorted) {
    const resultTokens = estimateTokens(result.content);
    if (tokenCount + resultTokens > MAX_TOKENS) break;
    
    selectedResults.push(result);
    tokenCount += resultTokens;
  }
  
  return selectedResults;
}
```

## Related Documentation

* [Knowledge](../Features/Knowledge.md) - Detailed guide on knowledge sources
* [Embeddings](../Features/Embeddings.md) - Learn about embedding models
* [Store](../Features/Store.md) - Storage options for embeddings
* [Tutorials/TelegramBotWithRAG.md](../Tutorials/TelegramBotWithRAG.md) - Complete RAG implementation example
* [Chain of Thought](Chain%20of%20Thought.md) - Combine RAG with reasoning for better results

## API Reference

```typescript
// RAG configuration options
interface RAGConfig {
  vectorStore: VectorStore;
  maxChunks?: number;
  similarityThreshold?: number;
  includeSources?: boolean;
  summarize?: boolean;
  reranker?: RerankerModel;
}

// Vector store interface
interface VectorStore {
  addDocuments(documents: string[], metadata?: any[]): Promise<void>;
  similaritySearch(query: string, k?: number, filter?: object): Promise<SearchResult[]>;
  delete(ids: string[]): Promise<void>;
}

// Search result interface
interface SearchResult {
  content: string;
  score: number;
  metadata?: Record<string, any>;
  id?: string;
}
```

---

*Last Updated: 2025-03-16*