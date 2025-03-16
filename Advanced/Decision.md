# Decision Making Framework

## Overview

The Decision Making framework in Alith enhances agent reasoning by using multiple independent reasoning paths and voting mechanisms to arrive at more robust and accurate decisions. This approach is particularly valuable for critical applications where reliability and confidence in the agent's output are essential. By combining Chain of Thought reasoning with statistical aggregation, Decision Making significantly reduces errors and improves consistency.

> **⚖️ Advanced Feature**: The Decision framework transforms single-pass reasoning into a deliberative process with multiple independent reasoning attempts, leading to more reliable and consistent results for mission-critical applications.

## Key Concepts

* **Multiple Reasoning Paths**: Independent reasoning attempts for the same problem
* **Voting Mechanisms**: Statistical methods to aggregate multiple reasoning results
* **Confidence Metrics**: Quantitative assessments of decision reliability
* **Dynamic Temperature**: Variation in creativity/randomness across reasoning attempts
* **Primitive Output Types**: Structured outputs like boolean, integer, or string values

## When to Use Decision Making

Decision Making is particularly valuable when:

* Your application requires high reliability and error reduction
* The stakes of incorrect decisions are significant
* You need quantifiable confidence metrics for agent outputs
* Problems benefit from multiple perspectives or reasoning approaches
* You need to handle problems with uncertain or ambiguous solutions

## Implementation Approaches

### Basic Decision Framework (Rust)

```rust
use alith::{
    llm_client::{prelude::*, DecisionTrait},
    LLM,
};
 
#[tokio::main]
async fn main() -> Result<(), anyhow::Error> {
    // Initialize the language model
    let model = LLM::from_model_name("gpt-4")?;
    let client = model.client();
    
    // Create a boolean decision request
    let response = client
        .reason()                  // Start with reasoning
        .boolean()                 // Use boolean output type
        .decision()                // Convert to a decision process
        .set_instructions("Is the sky blue?")
        .return_primitive()
        .await
        .unwrap();
    
    assert!(response);             // Returns true
 
    Ok(())
}
```

### Numeric Decision Making (Rust)

```rust
use alith::{
    llm_client::{prelude::*, DecisionTrait},
    LLM,
};
 
#[tokio::main]
async fn main() -> Result<(), anyhow::Error> {
    let model = LLM::from_model_name("gpt-4")?;
    let client = model.client();
    
    // Create an integer reasoning request with constraints
    let mut reason_request = client.reason().integer();
    reason_request.primitive.lower_bound(0).upper_bound(10);
    
    // Convert to a decision process
    let mut decision_request = reason_request.decision();
    
    // Execute the decision process
    let response = decision_request
        .set_instructions("How many fingers do you have?")
        .return_primitive()
        .await
        .unwrap();
    
    assert_eq!(response, 5);
    
    Ok(())
}
```

### Advanced Configuration (Rust)

```rust
use alith::{
    llm_client::{prelude::*, DecisionTrait},
    LLM,
};
 
#[tokio::main]
async fn main() -> Result<(), anyhow::Error> {
    let model = LLM::from_model_name("gpt-4")?;
    let client = model.client();
    
    // Set up an integer decision process
    let mut decision_request = client.reason().integer().decision();
    
    // Configure the decision process
    decision_request
        .best_of_n_votes(5)            // Use 5 independent reasoning attempts
        .dynamic_temperature(true);     // Vary temperature across attempts
 
    // Handle uncertain scenarios
    let response = decision_request
        .set_instructions("How many coins are in my pocket?")
        .return_optional_primitive()    // May return None for unknowable answers
        .await
        .unwrap();
    
    assert_eq!(response, None);         // Cannot know without more information
    
    Ok(())
}
```

### Node.js Implementation

```javascript
const { Agent, DecisionMaking } = require('alith');

async function demonstrateDecision() {
  // Create a base agent
  const agent = new Agent({
    name: "DecisionMaker",
    model: "gpt-4",
    preamble: "You are a careful decision maker who considers multiple perspectives."
  });

  // Create a boolean decision maker
  const booleanDecider = new DecisionMaking.Boolean({
    agent: agent,
    votingRounds: 5,              // Number of independent reasoning attempts
    dynamicTemperature: true,     // Vary temperature across attempts
    confidenceThreshold: 0.8      // Minimum confidence level required
  });
  
  // Make a boolean decision with confidence score
  const result = await booleanDecider.decide(
    "Based on historical data, will cryptocurrency prices rise next month?"
  );
  
  console.log(`Decision: ${result.value}`);
  console.log(`Confidence: ${result.confidence}`);
  console.log(`Reasoning: ${result.reasoning}`);
  
  // Create a numeric decision maker
  const numericDecider = new DecisionMaking.Integer({
    agent: agent,
    votingRounds: 7,
    lowerBound: 0, 
    upperBound: 10000,
    aggregationMethod: "median"   // Use median for numeric aggregation
  });
  
  // Make a numeric decision
  const priceResult = await numericDecider.decide(
    "What will be the price of Ethereum in USD at the end of the quarter?"
  );
  
  console.log(`Predicted price: $${priceResult.value}`);
  console.log(`Confidence interval: $${priceResult.confidenceInterval.min} - $${priceResult.confidenceInterval.max}`);
}

demonstrateDecision();
```

### Python Implementation

```python
from alith import Agent, DecisionMaking
import asyncio

async def demonstrate_decision():
    # Create a base agent
    agent = Agent(
        name="DecisionMaker",
        model="gpt-4",
        preamble="You are a careful decision maker who considers multiple perspectives."
    )
    
    # Create a string decision maker with allowed values
    string_decider = DecisionMaking.ExactString(
        agent=agent,
        allowed_values=["buy", "sell", "hold"],
        voting_rounds=5,
        confidence_threshold=0.7
    )
    
    # Make a decision from limited options
    result = await string_decider.decide(
        "Based on recent market trends, should an investor buy, sell, or hold Bitcoin?"
    )
    
    print(f"Recommendation: {result.value}")
    print(f"Confidence: {result.confidence}")
    print(f"Vote distribution: {result.vote_distribution}")
    
    # Create an integer decision maker with optional output
    integer_decider = DecisionMaking.Integer(
        agent=agent,
        voting_rounds=3,
        lower_bound=0,
        upper_bound=100,
        allow_none=True  # Can return None for unknowable answers
    )
    
    # Try to make a decision with insufficient information
    result = await integer_decider.decide(
        "How many transactions will occur on the Ethereum network in the next hour?"
    )
    
    if result.value is None:
        print("Decision: Unable to determine with current information")
    else:
        print(f"Estimated transactions: {result.value}")

# Run the async function
asyncio.run(demonstrate_decision())
```

## Common Patterns

### Pattern 1: Confidence-Based Fallback

Use confidence scores to determine whether to trust the decision:

```javascript
const { DecisionMaking } = require('alith');

async function confidentDecision(question) {
  const decider = new DecisionMaking.Boolean({
    agent: agent,
    votingRounds: 7,
    confidenceThreshold: 0.8
  });
  
  const result = await decider.decide(question);
  
  if (result.confidence >= 0.8) {
    // High confidence - use the automated decision
    return {
      value: result.value,
      automated: true,
      reasoning: result.reasoning
    };
  } else {
    // Low confidence - escalate to human review
    return {
      value: null,
      automated: false,
      reasoning: result.reasoning,
      status: "escalated_to_human"
    };
  }
}
```

### Pattern 2: Multi-Modal Decision Making

Combine different decision types for complex scenarios:

```python
from alith import Agent, DecisionMaking

async def investment_advisor(portfolio_data):
    agent = Agent(
        name="InvestmentAdvisor",
        model="gpt-4",
    )
    
    # First decision: Should we rebalance at all?
    rebalance_decider = DecisionMaking.Boolean(
        agent=agent,
        voting_rounds=5
    )
    
    rebalance_result = await rebalance_decider.decide(
        f"Based on this portfolio data, should we rebalance? {portfolio_data}"
    )
    
    if not rebalance_result.value:
        return {"action": "hold", "reasoning": rebalance_result.reasoning}
    
    # Second decision: What percentage for each asset class?
    allocation_decider = DecisionMaking.Json(
        agent=agent,
        voting_rounds=3,
        schema={
            "type": "object",
            "properties": {
                "stocks": {"type": "number", "minimum": 0, "maximum": 100},
                "bonds": {"type": "number", "minimum": 0, "maximum": 100},
                "crypto": {"type": "number", "minimum": 0, "maximum": 100},
                "cash": {"type": "number", "minimum": 0, "maximum": 100}
            },
            "required": ["stocks", "bonds", "crypto", "cash"]
        }
    )
    
    allocation_result = await allocation_decider.decide(
        f"Recommend percentage allocations for stocks, bonds, crypto, and cash: {portfolio_data}"
    )
    
    return {
        "action": "rebalance",
        "allocation": allocation_result.value,
        "confidence": allocation_result.confidence,
        "reasoning": allocation_result.reasoning
    }
```

## Configuration Options

| Option | Type | Description |
|--------|------|-------------|
| `votingRounds` | number | Number of independent reasoning attempts |
| `confidenceThreshold` | number | Minimum confidence level for decisions |
| `dynamicTemperature` | boolean | Whether to vary temperature across attempts |
| `aggregationMethod` | string | Method to aggregate results ("majority", "mean", "median") |
| `timeLimit` | number | Maximum time in ms for the entire decision process |
| `allowNone` | boolean | Whether to allow "unknowable" answers |
| `diversityBoost` | number | Factor to increase diversity in reasoning approaches |
| `lowerBound` | number | Lower bound for integer outputs |
| `upperBound` | number | Upper bound for integer outputs |
| `allowedValues` | string[] | Set of allowed string outputs |

## Best Practices

* **Appropriate Rounds**: Use more voting rounds for critical decisions
* **Confidence Thresholds**: Set confidence thresholds based on risk tolerance
* **Proper Bounds**: Define realistic bounds for numeric outputs
* **Time Management**: Balance thoroughness with time constraints
* **Fallback Mechanisms**: Implement graceful fallbacks for low-confidence decisions

> **⚠️ Warning:** While the Decision framework significantly improves reliability, it also increases token usage and API costs proportionally to the number of voting rounds. Balance accuracy needs with cost considerations.

## Common Issues and Solutions

### Issue 1: Inconsistent Results

Different runs produce significantly different outcomes.

**Solution**: Increase voting rounds and use deterministic settings:

```javascript
const decider = new DecisionMaking.Integer({
  agent: agent,
  votingRounds: 9,          // More rounds for stability
  dynamicTemperature: false, // Consistent temperature
  temperature: 0.2          // Low temperature for deterministic thinking
});
```

### Issue 2: Slow Performance

Decision making takes too long for interactive applications.

**Solution**: Implement timeouts and parallel execution:

```python
decider = DecisionMaking.Boolean(
    agent=agent,
    voting_rounds=5,
    time_limit=3000,         # 3 second limit
    parallel_execution=True  # Run reasoning attempts in parallel
)
```

## Advanced Usage

### Custom Aggregation Methods

Create specialized aggregation logic for domain-specific needs:

```javascript
const { DecisionMaking } = require('alith');

class CustomDecisionMaking extends DecisionMaking.Base {
  constructor(config) {
    super(config);
    this.riskTolerance = config.riskTolerance || "moderate";
  }
  
  async aggregateResults(results) {
    // Sort results by value
    results.sort((a, b) => a.value - b.value);
    
    // Apply risk tolerance to selection
    let selectedIdx;
    
    if (this.riskTolerance === "conservative") {
      // Pick from the lower range (25th percentile)
      selectedIdx = Math.floor(results.length * 0.25);
    } else if (this.riskTolerance === "aggressive") {
      // Pick from the upper range (75th percentile)
      selectedIdx = Math.floor(results.length * 0.75);
    } else {
      // Moderate - use median
      selectedIdx = Math.floor(results.length * 0.5);
    }
    
    return {
      value: results[selectedIdx].value,
      confidence: this.calculateConfidence(results),
      reasoning: results[selectedIdx].reasoning,
      distribution: this.getDistribution(results)
    };
  }
}

// Usage
const riskAdjustedDecider = new CustomDecisionMaking({
  agent: agent,
  votingRounds: 7,
  riskTolerance: "conservative"  // Custom parameter
});
```

## Related Documentation

* [Chain of Thought](Chain%20of%20Thought.md) - Understanding the reasoning that powers decisions
* [LLMs](../Features/LLMs.md) - Configure language models for optimal decision making
* [Advanced/Inference.md](Inference.md) - Optimize inference for decision workflows
* [Tutorials/MultiAgentSystem.md](../Tutorials/MultiAgentSystem.md) - Implement decisions in multi-agent systems

## API Reference

```typescript
// Decision Making interfaces
interface DecisionConfig {
  votingRounds?: number;
  confidenceThreshold?: number;
  dynamicTemperature?: boolean;
  aggregationMethod?: 'majority' | 'mean' | 'median' | 'custom';
  timeLimit?: number;
  allowNone?: boolean;
  diversityBoost?: number;
}

interface BooleanDecisionConfig extends DecisionConfig {
  // Boolean-specific options
}

interface IntegerDecisionConfig extends DecisionConfig {
  lowerBound?: number;
  upperBound?: number;
}

interface StringDecisionConfig extends DecisionConfig {
  allowedValues: string[];
}

// Decision result interface
interface DecisionResult<T> {
  value: T;
  confidence: number;
  reasoning: string;
  voteDistribution?: Record<string, number>;
  confidenceInterval?: { min: number; max: number };
}
```

---

*Last Updated: 2025-03-16*
