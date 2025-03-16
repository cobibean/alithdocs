# Chain of Thought (CoT)

## Overview

Chain of Thought (CoT) is an advanced prompting technique in Alith that encourages agents to break down complex problems into intermediate reasoning steps before arriving at a final answer. This approach significantly improves the accuracy and interpretability of responses, particularly for tasks requiring multi-step reasoning like mathematical problems, logical puzzles, or complex decision-making.

> **🧠 Advanced Feature**: Chain of Thought transforms your agents from black-box answer generators into transparent reasoning systems that show their work, improving both accuracy and explainability.

## Key Concepts

* **Reasoning Steps**: Explicit intermediate thoughts that break down a complex problem
* **Transparency**: Making the agent's thought process visible to users
* **Primitive Types**: Specific output formats like boolean, integer, or string values
* **Verification**: Using reasoning to verify outputs and catch logical errors
* **Optional Values**: Handling cases where a definitive answer isn't possible

## When to Use Chain of Thought

CoT is particularly valuable when:

* Your agent needs to solve complex, multi-step problems
* Users benefit from seeing the reasoning process, not just the answer
* You need to enforce specific output types (boolean, integer, etc.)
* You want to reduce errors by encouraging step-by-step analysis
* The problem requires careful logical evaluation before reaching a conclusion

## Implementation Approaches

### Primitive Reasoning (Rust)

The basic implementation of Chain of Thought in Rust:

```rust
use alith::{llm_client::prelude::*, LLM};
 
/// Enforces CoT style reasoning on the output of an LLM, before returning the requested primitive. Currently, reason is bound to the one_round reasoning workflow. Workflows relying on grammars are only supported by local LLMs.
#[tokio::main]
async fn main() -> Result<(), anyhow::Error> {
    let model = LLM::from_model_name("gpt-4")?;
    let client = model.client();
    // A boolean reason request
    let response = client
        .reason()
        .boolean()
        .set_instructions("Is the sky blue?")
        .return_primitive()
        .await
        .unwrap();
    assert!(response);
    // An integer reason request
    let mut reason_request = client.reason().integer();
    // Settings specific to the primitive can be accessed through the primitive field
    reason_request.primitive.lower_bound(0).upper_bound(10);
    let response = reason_request
        .set_instructions("How many fingers do you have?")
        .return_primitive()
        .await
        .unwrap();
    assert_eq!(response, 5);
 
    // Options
    let mut reason_request = client.reason().integer();
    // The conclusion and reasoning sentences can be set. This is useful for more complex reasoning tasks where you want the llm to pontificate more.
    reason_request
        .conclusion_sentences(4)
        .reasoning_sentences(3);
 
    // An integer request, but with an optional response
    let response = reason_request
        .set_instructions("How many coins are in my pocket?")
        .return_optional_primitive()
        .await
        .unwrap();
    assert_eq!(response, None);
 
    // An exact string reason request
    let mut reason_request = client.reason().exact_string();
    reason_request.primitive.add_string_to_allowed("red");
    reason_request
        .primitive
        .add_strings_to_allowed(&["blue", "green"]);
 
    let response = reason_request
        .set_instructions("What color is clorophyll?")
        .return_primitive()
        .await
        .unwrap();
    println!("{response}");
    Ok(())
}
```

### Customizing Reasoning (Rust)

You can customize the reasoning process:

```rust
use alith::{llm_client::prelude::*, LLM};

#[tokio::main]
async fn main() -> Result<(), anyhow::Error> {
    let model = LLM::from_model_name("gpt-4")?;
    let client = model.client();
    
    // Customize the reasoning process
    let mut reason_request = client.reason().integer();
    
    // Control the verbosity of reasoning
    reason_request
        .conclusion_sentences(4)  // More detailed conclusion
        .reasoning_sentences(3);  // More reasoning steps
 
    // Handle cases without a definitive answer
    let response = reason_request
        .set_instructions("How many coins are in my pocket?")
        .return_optional_primitive()  // May return None
        .await
        .unwrap();
    
    assert_eq!(response, None);  // Cannot determine without information
    
    Ok(())
}
```

### String Constraint Reasoning (Rust)

Limit string outputs to an allowed set of values:

```rust
use alith::{llm_client::prelude::*, LLM};

#[tokio::main]
async fn main() -> Result<(), anyhow::Error> {
    let model = LLM::from_model_name("gpt-4")?;
    let client = model.client();
    
    // Constrain output to specific strings
    let mut reason_request = client.reason().exact_string();
    
    // Add allowed values
    reason_request.primitive.add_string_to_allowed("red");
    reason_request.primitive.add_strings_to_allowed(&["blue", "green"]);
 
    // Force the agent to choose one of the allowed values
    let response = reason_request
        .set_instructions("What color is chlorophyll?")
        .return_primitive()
        .await
        .unwrap();
    
    println!("{}", response);  // Should output "green"
    
    Ok(())
}
```

### Node.js Implementation

```javascript
const { Agent, ChainOfThought } = require('alith');

async function demonstrateCoT() {
  // Create a math problem solver with CoT
  const agent = new Agent({
    name: "MathSolver",
    model: "gpt-4",
    preamble: "You are an expert mathematician who solves problems step-by-step.",
    chainOfThought: true  // Enable CoT by default
  });

  // Boolean reasoning
  const booleanSolver = new ChainOfThought.Boolean({
    agent: agent,
    reasoningSteps: 3  // Number of reasoning steps to encourage
  });
  
  const isPrime = await booleanSolver.solve("Is 97 a prime number?");
  console.log(`Is 97 prime? ${isPrime}`);
  
  // Integer reasoning
  const integerSolver = new ChainOfThought.Integer({
    agent: agent,
    lowerBound: 0,
    upperBound: 100,
    reasoningSteps: 3
  });
  
  const result = await integerSolver.solve("If I have 5 boxes with 7 items in each, how many items do I have total?");
  console.log(`Total items: ${result}`);
}

demonstrateCoT();
```

### Python Implementation

```python
from alith import Agent, ChainOfThought

async def demonstrate_cot():
    # Create a logical reasoning agent with CoT
    agent = Agent(
        name="LogicalReasoner",
        model="gpt-4",
        preamble="You are a logical reasoning expert who carefully analyzes problems step-by-step.",
        chain_of_thought=True  # Enable CoT by default
    )
    
    # Boolean reasoning
    boolean_solver = ChainOfThought.Boolean(
        agent=agent,
        reasoning_steps=3  # Encourage 3 reasoning steps
    )
    
    # Solve a logical puzzle
    valid = await boolean_solver.solve(
        "If all A are B, and some B are C, can we conclude that some A are C?"
    )
    print(f"Is the conclusion valid? {valid}")
    
    # String selection with constraints
    string_solver = ChainOfThought.ExactString(
        agent=agent,
        allowed_values=["mammals", "reptiles", "birds", "fish", "amphibians"]
    )
    
    # Force selection from allowed values
    classification = await string_solver.solve(
        "What class of vertebrates do dolphins belong to?"
    )
    print(f"Dolphins are {classification}")

# Run the async function
import asyncio
asyncio.run(demonstrate_cot())
```

## Common Patterns

### Pattern 1: Reasoning with Context

Provide additional context for more informed reasoning:

```javascript
const { ChainOfThought } = require('alith');

async function reasonWithContext() {
  const integerSolver = new ChainOfThought.Integer({
    agent: agent,
    lowerBound: 0,
    upperBound: 1000
  });
  
  // Add context to the reasoning process
  const result = await integerSolver.solve({
    question: "How many total calories are in the meal?",
    context: `
      Meal components:
      - Grilled chicken breast (6 oz): 210 calories
      - Brown rice (1 cup): 216 calories
      - Steamed broccoli (1 cup): 55 calories
      - Olive oil (1 tbsp): 119 calories
    `
  });
  
  console.log(`Total calories: ${result}`);
}
```

### Pattern 2: Multi-Stage Reasoning

Break complex problems into stages:

```python
from alith import Agent, ChainOfThought

async def multi_stage_reasoning(problem):
    agent = Agent(
        name="ComplexProblemSolver",
        model="gpt-4",
        chain_of_thought=True
    )
    
    # Stage 1: Problem analysis
    analysis = await agent.prompt(
        f"Analyze this problem and identify the key components: {problem}"
    )
    
    # Stage 2: Solution approach
    approach = await agent.prompt(
        f"Based on this analysis: {analysis}\nWhat approach should we take to solve it?"
    )
    
    # Stage 3: Mathematical reasoning
    solver = ChainOfThought.Integer(agent=agent)
    result = await solver.solve(
        f"Using this approach: {approach}\nSolve the original problem: {problem}"
    )
    
    return {
        "analysis": analysis,
        "approach": approach,
        "result": result
    }
```

## Configuration Options

| Option | Type | Description |
|--------|------|-------------|
| `reasoningSteps` | number | Number of reasoning steps to encourage |
| `conclusionSentences` | number | Number of sentences in the conclusion |
| `reasoningSentences` | number | Number of sentences in each reasoning step |
| `lowerBound` | number | Lower bound for integer outputs |
| `upperBound` | number | Upper bound for integer outputs |
| `allowedValues` | string[] | Allowed values for string outputs |
| `temperature` | number | Temperature for the reasoning process |
| `detailedOutput` | boolean | Whether to return detailed reasoning alongside the answer |

## Best Practices

* **Clear Instructions**: Provide clear, specific instructions for the reasoning task
* **Appropriate Bounds**: Set realistic bounds for numerical outputs
* **Verify Results**: Use assertions to verify outputs match expected types/ranges
* **Balance Verbosity**: Adjust reasoning and conclusion steps based on complexity
* **Optional Handling**: Use optional primitives when answers may not be determinable

> **💡 Tip:** For complex reasoning tasks, lower the temperature setting to encourage more deterministic, logical thinking.

## Common Issues and Solutions

### Issue 1: Inconsistent Reasoning

Agent's reasoning steps don't align with the final answer.

**Solution**: Increase reasoning steps and reduce temperature:

```javascript
const solver = new ChainOfThought.Integer({
  agent: agent,
  reasoningSteps: 5,  // More detailed reasoning
  temperature: 0.2    // More deterministic thinking
});
```

### Issue 2: Boundary Violations

Agent attempts to return values outside allowed range.

**Solution**: Use tighter constraints and explicit instructions:

```python
solver = ChainOfThought.Integer(
    agent=agent,
    lower_bound=0,
    upper_bound=100,
    preamble="You must return a number between 0 and 100 inclusive. If the true answer is outside this range, return the closest value within range."
)
```

## Advanced Usage

### Combining CoT with Decision Making

For critical applications, combine CoT with the Decision framework for more robust outcomes:

```rust
use alith::{llm_client::{prelude::*, DecisionTrait}, LLM};

#[tokio::main]
async fn main() -> Result<(), anyhow::Error> {
    let model = LLM::from_model_name("gpt-4")?;
    let client = model.client();
    
    // Create a reasoning request
    let mut reason_request = client.reason().integer();
    reason_request.primitive.lower_bound(0).upper_bound(1000);
    
    // Convert to a decision with multiple reasoning attempts
    let mut decision_request = reason_request.decision();
    decision_request
        .best_of_n_votes(5)           // Use 5 independent reasoning attempts
        .dynamic_temperature(true);    // Vary temperature across attempts
    
    // Solve a complex problem with multiple reasoning passes
    let response = decision_request
        .set_instructions("Calculate 1.5 * ((243 - 17) / 2) + 85")
        .return_primitive()
        .await
        .unwrap();
    
    println!("Final answer: {}", response);
    
    Ok(())
}
```

## Related Documentation

* [Decision](Decision.md) - Learn how to combine CoT with decision frameworks
* [LLMs](../Features/LLMs.md) - Configure language models for optimal reasoning
* [Retrieval-Augmented Generation (RAG)](Retrieval-Augmented%20Generation%20(RAG).md) - Combine reasoning with external knowledge
* [Tutorials/MultiAgentSystem.md](../Tutorials/MultiAgentSystem.md) - Use CoT in multi-agent systems

## API Reference

```typescript
// Chain of Thought interfaces
interface ChainOfThoughtConfig {
  reasoningSteps?: number;
  conclusionSentences?: number;
  reasoningSentences?: number;
  temperature?: number;
  detailedOutput?: boolean;
}

interface BooleanConfig extends ChainOfThoughtConfig {
  // Boolean-specific options
}

interface IntegerConfig extends ChainOfThoughtConfig {
  lowerBound?: number;
  upperBound?: number;
}

interface StringConfig extends ChainOfThoughtConfig {
  allowedValues: string[];
}

// Results interface with reasoning
interface ReasoningResult<T> {
  value: T;
  reasoning: string;
  confidence: number;
}
```

---

*Last Updated: 2025-03-16*