# Extractor - Structured Data Extraction

## Overview

The Extractor feature in Alith enables you to extract structured data from unstructured text using language models. This allows you to parse user inputs into well-defined data structures, making it easier to process information and perform operations based on user requests.

> **🔍 Core Feature**: Extractors transform unstructured natural language into structured data objects, enabling seamless integration with your application's type system.

## Key Concepts

* **Extraction Schema**: A structured definition of the data you want to extract
* **Type Safety**: Extracted data conforms to your defined types
* **Validation**: Ensuring extracted data meets your requirements
* **Inference**: Using LLMs to intelligently parse ambiguous text

## Usage

### Rust Implementation

```rust
use alith::{Extractor, LLM};
use schemars::JsonSchema;
use serde::{Deserialize, Serialize};
 
#[derive(Debug, Clone, Serialize, Deserialize, JsonSchema)]
struct Person {
    name: String,
    age: usize,
}
 
#[tokio::main]
async fn main() -> Result<(), anyhow::Error> {
    let model = LLM::from_model_name("gpt-4")?;
    let extractor = Extractor::new::<Person>(model);
    let response: Person = extractor.extract("Alice is 18 years old").await?;
    println!("{:?}", response); // Person { name: "Alice", age: 18 }
    Ok(())
}
```

### JavaScript Implementation

```javascript
const { Extractor } = require('alith');

// Define the structure to extract
const personSchema = {
  type: 'object',
  properties: {
    name: { type: 'string' },
    age: { type: 'number' }
  },
  required: ['name', 'age']
};

async function extractPerson() {
  const extractor = new Extractor({
    model: 'gpt-4',
    schema: personSchema
  });
  
  const result = await extractor.extract("Bob is 25 years old");
  console.log(result); // { name: "Bob", age: 25 }
}

extractPerson();
```

### Python Implementation

```python
from alith import Extractor
from pydantic import BaseModel
from typing import List, Optional

class Person(BaseModel):
    name: str
    age: int

async def extract_person():
    extractor = Extractor(model="gpt-4")
    result = await extractor.extract("Charlie is 30 years old", Person)
    print(result)  # Person(name="Charlie", age=30)

# Run the async function
import asyncio
asyncio.run(extract_person())
```

## Common Patterns

### Pattern 1: Complex Nested Structures

Extract hierarchical data from complex descriptions:

```rust
#[derive(Debug, Clone, Serialize, Deserialize, JsonSchema)]
struct Address {
    street: String,
    city: String,
    country: String,
    postal_code: Option<String>,
}

#[derive(Debug, Clone, Serialize, Deserialize, JsonSchema)]
struct Contact {
    email: String,
    phone: Option<String>,
}

#[derive(Debug, Clone, Serialize, Deserialize, JsonSchema)]
struct Employee {
    name: String,
    age: usize,
    position: String,
    address: Address,
    contact: Contact,
}

// Extract a complete employee record
let employee: Employee = extractor.extract(
    "David Smith is a 32-year-old software engineer living at 123 Main St, 
     San Francisco, USA, 94103. You can contact him at david@example.com or 555-123-4567."
).await?;
```

### Pattern 2: Multiple Entity Extraction

Extract multiple instances of the same structure:

```javascript
const reviewSchema = {
  type: 'array',
  items: {
    type: 'object',
    properties: {
      product: { type: 'string' },
      rating: { type: 'number', minimum: 1, maximum: 5 },
      comment: { type: 'string' }
    },
    required: ['product', 'rating']
  }
};

const reviews = await extractor.extract(
  "I bought the XPhone and would rate it 4/5, it's really good. Also got the YTablet 
   but it was disappointing, only 2/5 because the battery life is terrible.",
  reviewSchema
);

// reviews: [
//   { product: "XPhone", rating: 4, comment: "it's really good" },
//   { product: "YTablet", rating: 2, comment: "the battery life is terrible" }
// ]
```

## Configuration Options

| Option | Type | Description |
|--------|------|-------------|
| `model` | string | LLM model to use for extraction (e.g., "gpt-4") |
| `temperature` | number | Temperature for generation (lower for more deterministic results) |
| `maxRetries` | number | Number of extraction attempts before failing |
| `validationLevel` | string | Level of validation ("strict", "lenient", "none") |
| `formatInstructions` | string | Optional custom instructions for extraction |

## Best Practices

* **Clear Schemas**: Define clear and specific schemas for optimal extraction
* **Include Examples**: For complex structures, provide examples for better accuracy
* **Error Handling**: Implement robust error handling for extraction failures
* **Validation**: Always validate extracted data before using it in critical operations
* **Fallbacks**: Provide fallback values for optional fields when appropriate

> **💡 Tip:** For extracting complex structures, consider breaking down the extraction into multiple steps, extracting simpler sub-structures first and then combining them.

## Common Issues and Solutions

### Issue 1: Extraction Failures

The model fails to correctly extract the requested structure.

**Solution**: Improve extraction with better prompting and examples:

```rust
let extractor = Extractor::new::<Person>(model)
    .with_example(
        "John is 25 years old",
        Person { name: "John".to_string(), age: 25 }
    )
    .with_example(
        "Sarah, age 30, works as an engineer",
        Person { name: "Sarah".to_string(), age: 30 }
    );
```

### Issue 2: Type Mismatches

The extracted data doesn't match the expected types.

**Solution**: Use validation and type conversion:

```javascript
const ageExtractor = new Extractor({
  model: 'gpt-4',
  schema: {
    type: 'object',
    properties: {
      age: { type: 'number', minimum: 0, maximum: 120 }
    },
    required: ['age']
  },
  validation: 'strict',
  transformers: {
    // Convert string numbers to actual numbers
    age: (value) => typeof value === 'string' ? parseInt(value, 10) : value
  }
});
```

## Advanced Usage

### Custom Extraction with Specific Instructions

For specialized extraction tasks, provide detailed instructions:

```python
from alith import Extractor, ExtractionConfig
from pydantic import BaseModel
from typing import List

class MedicalCondition(BaseModel):
    condition: str
    symptoms: List[str]
    severity: str

extractor = Extractor(
    model="gpt-4",
    config=ExtractionConfig(
        system_prompt="""
        You are a medical data extraction assistant. Extract medical conditions, 
        symptoms, and severity from patient descriptions. Be precise and 
        follow medical terminology conventions. Severity should be categorized 
        as 'mild', 'moderate', or 'severe'.
        """,
        temperature=0.1,  # Low temperature for more consistent results
        max_retries=2
    )
)

result = await extractor.extract(
    "Patient reports recurring headaches and dizziness for the past week, 
     with moderate pain that increases when standing up quickly.",
    MedicalCondition
)
```

## Related Documentation

* [Tools](Tools.md) - Use extractors with tools for structured inputs
* [Advanced/Decision](../Advanced/Decision.md) - Combine extractors with decision-making
* [Features/LLMs](LLMs.md) - Configure language models for extraction
* [Tutorials/TelegramBot.md](../Tutorials/TelegramBot.md) - See extractors in a practical application

## API Reference

```typescript
// TypeScript interface for Extractor
interface ExtractionConfig {
  model: string;
  temperature?: number;
  maxRetries?: number;
  validationLevel?: 'strict' | 'lenient' | 'none';
  formatInstructions?: string;
  examples?: Array<{input: string, output: any}>;
}

class Extractor {
  constructor(config: ExtractionConfig);
  
  async extract<T>(input: string, schema: Schema | Class<T>): Promise<T>;
  
  addExample(input: string, output: any): Extractor;
  
  setValidationLevel(level: 'strict' | 'lenient' | 'none'): Extractor;
}
```

---

*Last Updated: 2025-03-16*