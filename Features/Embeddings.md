# Embeddings - Vector Representations for Semantic Search

## Overview

Embeddings in Alith are numerical representations of text that capture semantic meaning, enabling powerful capabilities like semantic search, clustering, and similarity comparisons. These vector representations form the foundation for knowledge retrieval, recommendation systems, and natural language understanding in your agent applications.

> **📊 Core Feature**: Embeddings transform text into mathematical vectors that preserve semantic relationships, allowing your agents to understand the meaning and context of information beyond simple keyword matching.

## Key Concepts

* **Embedding**: A vector (array of numbers) that represents a piece of text in a high-dimensional space
* **Embedding Model**: An AI model that converts text to embeddings, trained to place semantically similar texts closer together
* **Dimensionality**: The number of values in an embedding vector (typically 768-1536 dimensions)
* **Cosine Similarity**: A measure of similarity between vectors, often used to compare embeddings
* **Vector Database**: A specialized database optimized for storing and querying vector embeddings

## Usage

Alith provides multiple embedding model implementations that can be used with different AI providers or locally.

### OpenAI Embeddings (Rust)

Using OpenAI's embedding models for high-quality semantic representations:

```rust
use alith::{Agent, EmbeddingsBuilder, LLM};
 
#[tokio::main]
async fn main() -> Result<(), anyhow::Error> {
    // Initialize the model
    let model = LLM::from_model_name("gpt-4")?;
    
    // Create an embeddings model
    let embeddings_model = model.embeddings_model("text-embedding-3-small");
    
    // Generate embeddings for a set of documents
    let embeddings = EmbeddingsBuilder::new(embeddings_model.clone())
        .documents(vec![
            "Ethereum is a decentralized blockchain platform", 
            "Bitcoin is a cryptocurrency", 
            "Solana offers high transaction throughput"
        ])
        .unwrap()
        .build()
        .await?;
    
    println!("Generated {} embeddings", embeddings.len());
    
    Ok(())
}
```

### Local Fast Embeddings (Rust)

Running embedding models locally for privacy or offline usage:

```rust
use alith::{EmbeddingsBuilder, FastEmbeddingsModel};
 
#[tokio::main]
async fn main() -> Result<(), anyhow::Error> {
    // Initialize a local embedding model
    let embeddings_model = FastEmbeddingsModel::try_default().unwrap();
    
    // Generate embeddings for a set of documents
    let embeddings = EmbeddingsBuilder::new(embeddings_model.clone())
        .documents(vec![
            "Machine learning algorithms require data",
            "Neural networks have multiple layers",
            "Deep learning is a subset of machine learning"
        ])
        .unwrap()
        .build()
        .await?;
    
    println!("Embedding dimensions: {}", embeddings[0].len());
    
    Ok(())
}
```

### Node.js Embeddings Implementation

```javascript
const { Embeddings } = require('alith');

async function generateEmbeddings() {
  // Create an embeddings generator
  const embeddings = new Embeddings({
    model: "text-embedding-ada-002", // OpenAI model
    apiKey: process.env.OPENAI_API_KEY
  });

  // Generate embeddings for texts
  const vectors = await embeddings.generate([
    "Climate change is affecting global weather patterns",
    "Rising sea levels threaten coastal cities",
    "Renewable energy can help reduce carbon emissions"
  ]);

  console.log(`Generated ${vectors.length} embeddings`);
  console.log(`Each embedding has ${vectors[0].length} dimensions`);
  
  // Calculate similarity between first and second text
  const similarity = embeddings.calculateSimilarity(vectors[0], vectors[1]);
  console.log(`Similarity score: ${similarity}`);
}

generateEmbeddings();
```

### Python Embeddings Implementation

```python
from alith import Embeddings
import numpy as np

# Initialize embeddings with specified model
embeddings = Embeddings(model="text-embedding-3-small")

# Generate embeddings for a list of texts
texts = [
    "The patient shows symptoms of high fever",
    "Medical diagnosis requires careful examination",
    "Healthcare professionals need proper training"
]

vectors = embeddings.generate(texts)
print(f"Generated {len(vectors)} embeddings of dimension {len(vectors[0])}")

# Calculate cosine similarity between vectors
def cosine_similarity(v1, v2):
    return np.dot(v1, v2) / (np.linalg.norm(v1) * np.linalg.norm(v2))

similarity = cosine_similarity(vectors[0], vectors[2])
print(f"Similarity between text 1 and 3: {similarity}")
```

## Common Patterns

### Pattern 1: Semantic Search Implementation

Use embeddings to build a semantic search system:

```javascript
const { Embeddings, VectorStore } = require('alith');

async function createSemanticSearch() {
  // Create embeddings generator
  const embeddings = new Embeddings({
    model: "text-embedding-ada-002"
  });

  // Create a vector store with documents
  const documents = [
    "Alith is a framework for building AI agents",
    "Agents can use tools to interact with external systems",
    "Knowledge bases enhance agents with domain-specific information",
    "Memory allows agents to maintain conversation context",
    "Web3 integration connects agents to blockchain data"
  ];

  // Initialize the vector store
  const vectorStore = new VectorStore({
    embeddings: embeddings,
    documents: documents
  });

  // Search for semantically similar documents
  const results = await vectorStore.similaritySearch(
    "How can I build an AI assistant that remembers conversations?",
    2  // Return top 2 results
  );

  console.log("Search results:", results);
}

createSemanticSearch();
```

### Pattern 2: Document Clustering

Group similar documents using embeddings:

```python
from alith import Embeddings
from sklearn.cluster import KMeans
import numpy as np

# Generate embeddings for documents
embeddings_model = Embeddings(model="text-embedding-3-small")
documents = [
    "Bitcoin price surges to new all-time high",
    "Ethereum completes major network upgrade",
    "New cryptocurrency regulation proposed in EU",
    "Stock market shows signs of recovery",
    "Tech companies report strong quarterly earnings",
    "Investment banks predict market correction"
]

# Generate embeddings
vectors = embeddings_model.generate(documents)

# Perform clustering
kmeans = KMeans(n_clusters=2, random_state=42)
clusters = kmeans.fit_predict(vectors)

# Group documents by cluster
grouped_docs = {}
for i, cluster_id in enumerate(clusters):
    if cluster_id not in grouped_docs:
        grouped_docs[cluster_id] = []
    grouped_docs[cluster_id].append(documents[i])

# Print clusters
for cluster_id, docs in grouped_docs.items():
    print(f"Cluster {cluster_id}:")
    for doc in docs:
        print(f"  - {doc}")
```

## Configuration Options

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `model` | string | Varies by SDK | Embedding model to use |
| `dimensions` | number | Model-dependent | Number of dimensions in output vectors |
| `apiKey` | string | From env | API key for service (OpenAI, etc.) |
| `batchSize` | number | 20 | Number of texts to embed in each API call |
| `maxRetries` | number | 3 | Number of times to retry failed API calls |
| `timeout` | number | 60000 | Timeout in milliseconds for API calls |
| `local` | boolean | false | Whether to use local models |
| `normalize` | boolean | true | Whether to normalize vectors |

## Best Practices

* **Right-Sized Embeddings**: Choose appropriate embedding models based on your accuracy needs and performance constraints
* **Batching**: Process embeddings in batches for better efficiency with API calls
* **Caching**: Cache embedding results for frequently used text to reduce API costs
* **Text Preprocessing**: Clean and normalize text before generating embeddings
* **Dimensionality**: Higher dimensional embeddings preserve more information but require more storage

> **💡 Tip:** For large document collections, consider chunking documents into smaller segments before embedding to improve retrieval precision.

## Common Issues and Solutions

### Issue 1: Embedding API Rate Limits

Hitting rate limits when generating large numbers of embeddings.

**Solution**: Implement batch processing with appropriate delays between batches:

```javascript
async function generateWithRateLimiting(texts) {
  const batchSize = 100;
  const delayMs = 1000;
  const embeddings = new Embeddings({ model: "text-embedding-ada-002" });
  
  const results = [];
  
  for (let i = 0; i < texts.length; i += batchSize) {
    const batch = texts.slice(i, i + batchSize);
    console.log(`Processing batch ${i/batchSize + 1}/${Math.ceil(texts.length/batchSize)}`);
    
    const batchResults = await embeddings.generate(batch);
    results.push(...batchResults);
    
    if (i + batchSize < texts.length) {
      console.log(`Waiting ${delayMs}ms to avoid rate limits...`);
      await new Promise(resolve => setTimeout(resolve, delayMs));
    }
  }
  
  return results;
}
```

### Issue 2: Embedding Quality for Specialized Domains

Generic embeddings don't capture domain-specific semantics well.

**Solution**: Use domain-specific embedding models or fine-tune embeddings:

```python
from alith import Embeddings

# Domain-specific embeddings
legal_embeddings = Embeddings(
    model="text-embedding-3-large",  # Higher quality model for specialized domains
    normalize=True,
    tokenization_options={
        "preserve_case": True,       # Legal text may have case-significant terms
        "preserve_punctuation": True # Important in legal documents
    }
)

legal_texts = [
    "The party of the first part hereby agrees...",
    "Subject to the provisions of Section 3.2...",
    "The defendant shall make restitution..."
]

vectors = legal_embeddings.generate(legal_texts)
```

## Advanced Usage

### Custom Embedding Model Integration

Integrate specialized embedding models for specific domains:

```javascript
const { BaseEmbeddingsModel } = require('alith');
const axios = require('axios');

class CustomEmbeddingsModel extends BaseEmbeddingsModel {
  constructor(apiEndpoint, apiKey) {
    super();
    this.apiEndpoint = apiEndpoint;
    this.apiKey = apiKey;
  }
  
  async generateEmbeddings(texts) {
    try {
      const response = await axios.post(this.apiEndpoint, {
        texts: texts
      }, {
        headers: {
          'Authorization': `Bearer ${this.apiKey}`,
          'Content-Type': 'application/json'
        }
      });
      
      return response.data.embeddings;
    } catch (error) {
      console.error('Error generating embeddings:', error);
      throw new Error(`Embedding generation failed: ${error.message}`);
    }
  }
}

// Usage
const customModel = new CustomEmbeddingsModel(
  'https://api.customservice.com/embed',
  process.env.CUSTOM_API_KEY
);

const embeddings = new Embeddings({
  model: customModel
});
```

## Related Documentation

* [Knowledge](Knowledge.md) - See how embeddings power knowledge retrieval
* [Store](Store.md) - Learn about vector stores for embedding persistence
* [LLMs](LLMs.md) - Compare embedding models with LLM capabilities
* [Tutorial: Building a RAG-based Telegram Bot](../Tutorials/TelegramBotWithRAG.md) - See embeddings in action for retrieval

## API Reference

```typescript
// Embeddings interfaces
interface EmbeddingsConfig {
  model: string | EmbeddingsModel;
  apiKey?: string;
  dimensions?: number;
  batchSize?: number;
  maxRetries?: number;
  timeout?: number;
  normalize?: boolean;
}

interface EmbeddingsModel {
  generateEmbeddings(texts: string[]): Promise<number[][]>;
  getDimensions(): number;
  getModelName(): string;
}

// Vector operations
interface VectorOperations {
  calculateSimilarity(v1: number[], v2: number[]): number;
  normalizeVector(vector: number[]): number[];
}
```

---

*Last Updated: 2025-03-16*