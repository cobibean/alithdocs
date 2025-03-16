# Store - Persistent Data Management for Alith

## Overview

The Store feature in Alith allows you to persist and retrieve data across sessions or interactions. This is essential for maintaining information between agent invocations, storing user preferences, caching expensive computation results, or creating knowledge bases that can be queried efficiently for retrieval-augmented generation (RAG).

> **💾 Core Feature**: The Store system enables your agents to maintain state across restarts and sessions, creating persistent experiences and knowledge repositories that grow over time.

## Key Concepts

* **Storage Backend**: Where data is physically stored (in-memory, vector database, etc.)
* **Embeddings**: Vector representations of text used for semantic search in stores
* **Document Storage**: Ability to store and retrieve text documents
* **Vector Search**: Finding semantically similar items in a store
* **Persistence**: Maintaining data across application restarts
* **Collection**: A logical grouping of related items in a store

## Usage

Alith provides multiple storage implementations to suit different use cases, from simple in-memory options to sophisticated vector databases.

### In-Memory Storage (Rust)

For temporary storage that doesn't persist beyond application lifetime:

```rust
use alith::{Agent, InMemoryStorage, LLM};
 
#[tokio::main]
async fn main() -> Result<(), anyhow::Error> {
    // Initialize model and get embeddings model
    let model = LLM::from_model_name("gpt-4")?;
    let embeddings_model = model.embeddings_model("text-embedding-3-small");
    
    // Create in-memory storage
    let storage = InMemoryStorage::from_multiple_documents::<()>(
        embeddings_model, 
        vec![
            ("What is glarb-glarb?", "Glarb-glarb is a fictional term used as a placeholder in examples."),
            ("What is blorp?", "Blorp is a made-up word often used to demonstrate storage functionality.")
        ]
    );
    
    // Create agent with store
    let agent = Agent::new("dictionary assistant", model, vec![])
        .preamble(
            r#"You are a dictionary assistant here to assist the user in understanding the meaning of words.
You will find additional non-standard word definitions that could be useful below."#
        )
        .store_index(1, storage);
    
    // Interact with the agent
    let response = agent.prompt("What does \"glarb-glarb\" mean?").await?;
    println!("{}", response);
    
    Ok(())
}
```

### Qdrant Vector Database (Rust)

For persistent storage and efficient semantic search at scale:

```rust
use alith::store::qdrant::{
    CreateCollectionBuilder, Distance, QdrantClient, QdrantStorage, VectorParamsBuilder,
    DEFAULT_COLLECTION_NAME,
};
use alith::{Agent, EmbeddingsBuilder, LLM};
 
#[tokio::main]
async fn main() -> Result<(), anyhow::Error> {
    // Initialize model and embeddings model
    let model = LLM::from_model_name("gpt-4")?;
    let embeddings_model = model.embeddings_model("text-embedding-3-small");
    
    // Connect to Qdrant database
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
    
    // Create Qdrant storage with documents
    let storage = QdrantStorage::from_multiple_documents::<()>(
        client, 
        embeddings_model, 
        vec![
            ("What is a blockchain?", "A blockchain is a distributed database or ledger shared among network nodes that stores data in blocks linked together via cryptography."),
            ("What is a smart contract?", "A smart contract is a self-executing program stored on a blockchain that runs when predetermined conditions are met.")
        ]
    ).await?;
    
    // Create agent with store
    let agent = Agent::new("blockchain assistant", model, vec![])
        .preamble(
            r#"You are a blockchain expert who helps users understand crypto terminology.
You will find additional blockchain definitions that could be useful below."#
        )
        .store_index(1, storage);
    
    // Interact with the agent
    let response = agent.prompt("Can you explain what a smart contract is?").await?;
    println!("{}", response);
    
    Ok(())
}
```

### Node.js Store Implementation

```javascript
const { Agent, Store } = require('alith');

async function createAgentWithStore() {
  // Initialize in-memory store with documents
  const store = new Store.InMemory({
    embeddingModel: "text-embedding-ada-002",
    documents: [
      {
        content: "The annual shareholder meeting will be held on June 15, 2023 at 2:00 PM EST.",
        metadata: { type: "meeting", year: 2023 }
      },
      {
        content: "Q1 revenue was $10.2M, a 15% increase over the same period last year.",
        metadata: { type: "financial", period: "Q1" }
      },
      {
        content: "The board approved a new sustainability initiative to reduce carbon emissions by 30% by 2025.",
        metadata: { type: "initiative", category: "sustainability" }
      }
    ]
  });

  // Create agent with store
  const agent = new Agent({
    name: "CompanyInfoAgent",
    model: "gpt-4",
    preamble: "You are a corporate information assistant who helps with company data and documents.",
    stores: [store]
  });

  // Query the agent
  const response = await agent.prompt(
    "When is the annual shareholder meeting this year?"
  );
  
  console.log(response);
  
  // Add new document to store
  await store.addDocuments([
    {
      content: "The Q2 earnings call is scheduled for August 3, 2023 at 11:00 AM EST.",
      metadata: { type: "meeting", period: "Q2" }
    }
  ]);
  
  // Query with the updated store
  const response2 = await agent.prompt(
    "When is the Q2 earnings call?"
  );
  
  console.log(response2);
}

createAgentWithStore();
```

### Python Store Implementation

```python
from alith import Agent, Store
import os

# Initialize a vector database store
async def create_agent_with_pinecone_store():
    # Create the store
    pinecone_store = Store.Pinecone(
        api_key=os.environ["PINECONE_API_KEY"],
        environment="us-west1-gcp",
        index_name="company-knowledge",
        embedding_model="text-embedding-3-small",
        namespace="legal-documents"
    )
    
    # Add documents to the store
    await pinecone_store.add_documents([
        {
            "content": "Privacy Policy: Users must be informed of data collection practices.",
            "metadata": {"doc_type": "policy", "department": "legal"}
        },
        {
            "content": "Terms of Service: The service is provided 'as-is' with no warranties.",
            "metadata": {"doc_type": "terms", "department": "legal"}
        },
        {
            "content": "GDPR Compliance: Users can request deletion of their personal data.",
            "metadata": {"doc_type": "compliance", "department": "legal"}
        }
    ])
    
    # Create agent with the store
    agent = Agent(
        name="LegalAssistant",
        model="gpt-4",
        preamble="You are a legal assistant who helps with company policies and compliance.",
        stores=[pinecone_store]
    )
    
    # Query the agent
    response = await agent.prompt(
        "What are our GDPR compliance requirements?"
    )
    
    print(response)
    
    # Search the store directly
    results = await pinecone_store.similarity_search(
        "data deletion rights",
        k=2,  # Return top 2 matches
        filter={"department": "legal"}
    )
    
    print("Direct store search results:")
    for doc in results:
        print(f"- {doc.content} (Score: {doc.score})")
```

## Common Patterns

### Pattern 1: Retrieval Augmented Generation (RAG)

Use store to enhance agent responses with relevant information:

```javascript
const { Agent, Store, Tools } = require('alith');

async function createRAGAgent() {
  // Create document store from a PDF
  const pdfStore = await Store.fromPDF({
    filePath: "./data/product_manual.pdf",
    chunkSize: 1000,
    chunkOverlap: 200,
    embeddingModel: "text-embedding-ada-002"
  });
  
  // Create a search tool that queries the store
  const searchTool = Tools.createStorageSearchTool({
    name: "searchManual",
    description: "Search the product manual for information",
    store: pdfStore
  });
  
  // Create agent with store and search tool
  const agent = new Agent({
    name: "ProductSupport",
    model: "gpt-4",
    preamble: `You are a product support specialist who helps users with questions about our product.
Use the searchManual tool to find relevant information in the product manual when needed.`,
    tools: [searchTool]
  });
  
  // User query
  const response = await agent.prompt(
    "How do I reset my device to factory settings?"
  );
  
  console.log(response);
}

createRAGAgent();
```

### Pattern 2: Multi-Store Configuration

Combine different stores for different types of knowledge:

```python
from alith import Agent, Store

async def create_multi_store_agent():
    # Technical documentation in a vector database
    tech_docs = Store.Chroma(
        collection_name="technical_docs",
        embedding_model="text-embedding-3-small",
        persist_directory="./data/chroma_db"
    )
    
    # FAQ in an in-memory store for quick access
    faq_store = Store.InMemory(
        documents=[
            {"content": "Q: How do I reset my password? A: Use the 'Forgot Password' link on the login page."},
            {"content": "Q: What payment methods do you accept? A: We accept Visa, Mastercard, and PayPal."},
            {"content": "Q: How do I cancel my subscription? A: Go to Account Settings and select 'Manage Subscription'."}
        ],
        embedding_model="text-embedding-3-small"
    )
    
    # Create agent with multiple stores
    agent = Agent(
        name="SupportBot",
        model="gpt-4",
        preamble="""You are a customer support agent. You have access to two knowledge sources:
1. Technical documentation (for detailed product information)
2. FAQ (for common customer questions)
Use these sources to provide accurate answers to customer questions.""",
        stores=[
            {"name": "technical_docs", "store": tech_docs},
            {"name": "faq", "store": faq_store}
        ]
    )
    
    response = await agent.prompt("How can I cancel my subscription?")
    print(response)
```

## Configuration Options

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `type` | string | `'memory'` | Storage type (memory, qdrant, pinecone, etc.) |
| `embeddingModel` | string | `'text-embedding-ada-002'` | Model used for embeddings |
| `chunkSize` | number | 1000 | Size of text chunks for processing large documents |
| `chunkOverlap` | number | 200 | Overlap between chunks to maintain context |
| `similarity` | string | `'cosine'` | Similarity measure (cosine, euclidean, dot) |
| `maxResults` | number | 5 | Maximum number of results to return from searches |
| `scoreThreshold` | number | 0.7 | Minimum similarity score to include in results |
| `connectionString` | string | - | Database connection string (for external DBs) |

## Best Practices

* **Chunk Appropriately**: When storing large documents, choose appropriate chunk sizes
* **Index Strategically**: Only store information that's valuable for retrieval
* **Use Metadata**: Add metadata to documents for effective filtering
* **Consider Persistence**: Choose a persistent store for production applications
* **Refresh Data**: Update store contents when source information changes

> **⚠️ Warning:** In-memory stores do not persist data after application restarts. For production applications requiring persistence, use database-backed stores.

## Common Issues and Solutions

### Issue 1: Slow Query Performance

Vector search operations taking too long, especially with large stores.

**Solution**: Optimize index configurations and use filtering:

```javascript
// Create store with optimized parameters
const store = new Store.Pinecone({
  // ...configuration
  dimension: 1536,  // Match embedding model dimension
  metric: "cosine",
  podType: "p1", // Higher performance pod
  indexing: {
    replicas: 1,
    shards: 2  // Better for larger indexes
  }
});

// Use metadata filtering for more efficient searches
const results = await store.similaritySearch(
  "security requirements",
  {
    filter: { 
      category: "compliance",
      year: { $gte: 2022 }  // Only recent documents
    },
    maxResults: 3
  }
);
```

### Issue 2: Embedding API Costs

High costs from generating embeddings for large document collections.

**Solution**: Implement caching and batching:

```python
from alith import Store
import hashlib
import json
import os

class CachedEmbeddingStore:
    def __init__(self, base_store, cache_dir="./embeddings_cache"):
        self.base_store = base_store
        self.cache_dir = cache_dir
        os.makedirs(cache_dir, exist_ok=True)
    
    def _get_cache_path(self, text):
        # Create a hash of the text to use as filename
        text_hash = hashlib.md5(text.encode()).hexdigest()
        return os.path.join(self.cache_dir, f"{text_hash}.json")
    
    async def add_document(self, content, metadata=None):
        cache_path = self._get_cache_path(content)
        
        # Check if embedding is already cached
        if os.path.exists(cache_path):
            print(f"Using cached embedding for document")
            with open(cache_path, 'r') as f:
                embedding_data = json.load(f)
                # Use cached embedding directly with store's internal methods
                await self.base_store._add_embedding(
                    embedding_data['embedding'],
                    content,
                    metadata
                )
        else:
            # Generate new embedding and cache it
            embedding = await self.base_store.add_document(content, metadata)
            
            # Save to cache
            with open(cache_path, 'w') as f:
                json.dump({
                    'embedding': embedding,
                    'content': content,
                    'metadata': metadata
                }, f)
            
            return embedding
```

## Advanced Usage

### Custom Storage Implementation

Create a specialized storage backend for unique requirements:

```javascript
const { BaseStore } = require('alith');
const redis = require('redis');

class RedisStore extends BaseStore {
  constructor(options) {
    super();
    this.client = redis.createClient(options.redisUrl);
    this.namespace = options.namespace || 'alith:store';
    this.embeddingModel = options.embeddingModel || 'text-embedding-ada-002';
    this.embeddings = new Embeddings({ model: this.embeddingModel });
  }
  
  async initialize() {
    await this.client.connect();
  }
  
  async addDocument(content, metadata = {}) {
    // Generate embedding
    const embedding = await this.embeddings.generate([content])[0];
    
    // Create document ID
    const docId = `doc:${Date.now()}:${Math.random().toString(36).substring(2, 9)}`;
    
    // Store document content
    await this.client.hSet(`${this.namespace}:docs`, docId, JSON.stringify({
      content,
      metadata,
      timestamp: Date.now()
    }));
    
    // Store embedding with Redis HSET
    await this.client.hSet(`${this.namespace}:embeddings`, docId, JSON.stringify(embedding));
    
    return docId;
  }
  
  async similaritySearch(query, options = {}) {
    const k = options.k || 5;
    
    // Generate query embedding
    const queryEmbedding = await this.embeddings.generate([query])[0];
    
    // Get all document embeddings
    const allEmbeddings = await this.client.hGetAll(`${this.namespace}:embeddings`);
    
    // Calculate similarity scores
    const scores = Object.entries(allEmbeddings).map(([docId, embeddingStr]) => {
      const embedding = JSON.parse(embeddingStr);
      const similarity = this.cosineSimilarity(queryEmbedding, embedding);
      return { docId, similarity };
    });
    
    // Sort by similarity and take top k
    const topResults = scores
      .sort((a, b) => b.similarity - a.similarity)
      .slice(0, k);
    
    // Get document content for top results
    const results = await Promise.all(
      topResults.map(async ({ docId, similarity }) => {
        const docJson = await this.client.hGet(`${this.namespace}:docs`, docId);
        const { content, metadata } = JSON.parse(docJson);
        return { docId, content, metadata, score: similarity };
      })
    );
    
    return results;
  }
  
  // Helper method to calculate cosine similarity
  cosineSimilarity(vecA, vecB) {
    const dotProduct = vecA.reduce((sum, val, i) => sum + val * vecB[i], 0);
    const magA = Math.sqrt(vecA.reduce((sum, val) => sum + val * val, 0));
    const magB = Math.sqrt(vecB.reduce((sum, val) => sum + val * val, 0));
    return dotProduct / (magA * magB);
  }
  
  async close() {
    await this.client.disconnect();
  }
}

// Usage
const redisStore = new RedisStore({
  redisUrl: 'redis://localhost:6379',
  namespace: 'product:kb',
  embeddingModel: 'text-embedding-ada-002'
});

await redisStore.initialize();
await redisStore.addDocument("Product warranty covers manufacturing defects for 2 years.");
```

## Related Documentation

* [Knowledge](Knowledge.md) - How Store relates to Knowledge sources
* [Embeddings](Embeddings.md) - Learn about the embedding models used by Store
* [Memory](Memory.md) - Compare memory (conversation history) with persistent storage
* [Tutorial: Building a RAG-based Telegram Bot](../Tutorials/TelegramBotWithRAG.md) - See Store in action for retrieval

## API Reference

```typescript
// Store interfaces
interface StoreConfig {
  type: 'memory' | 'qdrant' | 'pinecone' | 'chroma' | 'custom';
  embeddingModel: string;
  chunkSize?: number;
  chunkOverlap?: number;
  similarity?: 'cosine' | 'euclidean' | 'dot';
  maxResults?: number;
  scoreThreshold?: number;
  connectionString?: string;
}

interface StorageBackend {
  addDocument(content: string, metadata?: Record<string, any>): Promise<string>;
  addDocuments(documents: Array<{content: string, metadata?: Record<string, any>}>): Promise<string[]>;
  similaritySearch(query: string, options?: SearchOptions): Promise<SearchResult[]>;
  deleteDocument(id: string): Promise<boolean>;
  getDocument(id: string): Promise<Document | null>;
}

interface SearchOptions {
  k?: number;
  filter?: Record<string, any>;
  scoreThreshold?: number;
}

interface SearchResult {
  docId: string;
  content: string;
  metadata?: Record<string, any>;
  score: number;
}
```

---

*Last Updated: 2025-03-16*