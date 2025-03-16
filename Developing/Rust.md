# Developing with Rust

## Overview

This guide provides instructions for setting up a development environment for the Alith SDK using Rust. Whether you're contributing to the core Alith project or building your own agent applications with the Rust SDK, this document will help you get started with the necessary tools and workflows.

> **🛠️ Development Environment**: The Rust SDK offers high-performance, memory-safe implementation of Alith agents with full access to all framework features.

## Prerequisites

Before you begin developing with the Rust SDK, ensure you have the following:

* **Rust Toolchain**: Version 1.70.0 or later
* **Cargo**: Rust's package manager (installed with Rust)
* **Git**: For version control
* **Optional**: An IDE with Rust support (VS Code with rust-analyzer, IntelliJ with Rust plugin, etc.)

## Setup Instructions

### Installing Rust

If you don't have Rust installed, you can install it using rustup:

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

After installation, verify that Rust is properly installed:

```bash
rustc --version
# Expected output: rustc x.y.z (abcdefg 20XX-XX-XX)
```

### Setting Up the Project

#### Option 1: Contributing to Alith SDK

If you're planning to contribute to the Alith SDK:

```bash
# Clone the repository
git clone https://github.com/0xLazAI/alith.git
cd alith

# Build the project
make check

# Run tests
make test
```

#### Option 2: Starting a New Project with Alith

To create a new Rust project that uses Alith:

```bash
# Create a new Rust project
cargo new my_alith_project
cd my_alith_project

# Add Alith as a dependency in Cargo.toml
echo '[dependencies]
alith = "0.1.0"' >> Cargo.toml

# Create a basic agent example
cat > src/main.rs << EOL
use alith::{Agent, Tool};

fn main() {
    // Define a simple tool
    let weather_tool = Tool::new()
        .name("get_weather")
        .description("Get the current weather for a location")
        .schema(r#"{
            "type": "object",
            "properties": {
                "location": {
                    "type": "string",
                    "description": "The city and state/country"
                }
            },
            "required": ["location"]
        }"#)
        .function(|args| {
            let location = args["location"].as_str().unwrap_or("unknown");
            Ok(format!("The weather in {} is sunny and 72°F", location))
        });

    // Create an agent with the tool
    let agent = Agent::new()
        .name("WeatherAssistant")
        .model("gpt-4")
        .api_key(std::env::var("OPENAI_API_KEY").unwrap_or_default())
        .preamble("You are a helpful assistant that provides weather information.")
        .add_tool(weather_tool);

    // Prompt the agent
    let response = agent.prompt("What's the weather like in San Francisco?");
    println!("Agent response: {}", response);
}
EOL

# Build and run
cargo build
```

## Development Workflow

### Building the Project

To build the Alith SDK or your Rust project using Alith:

```bash
cargo build
```

For a release build with optimizations:

```bash
cargo build --release
```

### Running Tests

Run the test suite to ensure everything is working correctly:

```bash
cargo test
```

Or with the Makefile (for Alith SDK development):

```bash
make test
```

### Code Formatting and Linting

Ensure your code follows Rust's formatting standards:

```bash
# Format code
cargo fmt
# or with the Makefile
make fmt

# Run linter
cargo clippy
# or with the Makefile
make clippy
```

## Advanced Topics

### Working with Features

Alith's Rust SDK supports feature flags to enable or disable certain functionalities:

```toml
# In Cargo.toml
[dependencies]
alith = { version = "0.1.0", features = ["web3", "embeddings", "memory"] }
```

Available features include:

| Feature | Description |
|---------|-------------|
| `web3` | Enables blockchain integration tools |
| `embeddings` | Adds support for vector embeddings |
| `memory` | Includes memory management capabilities |
| `full` | Enables all features |

### Creating Custom Tool Implementations

Create more sophisticated tools by implementing the `Tool` trait:

```rust
use alith::{Tool, ToolContext, ToolResult};
use serde_json::Value;

struct DatabaseTool;

impl Tool for DatabaseTool {
    fn name(&self) -> &str {
        "query_database"
    }

    fn description(&self) -> &str {
        "Run a query against the database"
    }

    fn schema(&self) -> &str {
        r#"{
            "type": "object",
            "properties": {
                "query": {
                    "type": "string",
                    "description": "SQL query to execute"
                }
            },
            "required": ["query"]
        }"#
    }

    fn execute(&self, args: Value, _context: &ToolContext) -> ToolResult {
        let query = args["query"].as_str().unwrap_or("");
        // In a real implementation, you would connect to a database here
        println!("Executing query: {}", query);
        Ok(Value::String(format!("Query executed: {}", query)))
    }
}

// Usage
let agent = Agent::new()
    .name("DatabaseAgent")
    .model("gpt-4")
    .api_key("YOUR_API_KEY")
    .add_tool(Box::new(DatabaseTool));
```

## Best Practices

* **Error Handling**: Use Rust's Result type effectively for robust error handling in tools
* **Memory Management**: Leverage Rust's ownership system to create efficient agents
* **Async Support**: Use async/await for non-blocking operations, especially for API calls
* **Type Safety**: Take advantage of Rust's strong type system for safer code
* **Testing**: Write unit tests for custom tools and agent behaviors

> **💡 Tip:** When creating tools that interact with external services, consider using the `async_trait` crate to implement asynchronous tool execution methods.

## Common Issues and Solutions

### Issue 1: API Key Configuration

**Problem**: Unable to authenticate with LLM providers.

**Solution**: Ensure your API key is properly set:

```rust
// Either through environment variables
let api_key = std::env::var("OPENAI_API_KEY").expect("OPENAI_API_KEY not set");

// Or directly (not recommended for production)
let agent = Agent::new()
    .model("gpt-4")
    .api_key("your-api-key");

// Better approach using dotenv for development
use dotenv::dotenv;
dotenv().ok();
let api_key = std::env::var("OPENAI_API_KEY").expect("OPENAI_API_KEY not set");
```

### Issue 2: Serialization/Deserialization Errors

**Problem**: JSON serialization issues with tool inputs/outputs.

**Solution**: Use proper error handling with serde:

```rust
fn execute(&self, args: Value, _context: &ToolContext) -> ToolResult {
    let query = match args.get("query").and_then(|v| v.as_str()) {
        Some(q) => q,
        None => return Err("Missing or invalid 'query' parameter".into()),
    };
    
    // Process query...
    Ok(serde_json::json!({
        "status": "success",
        "results": [
            {"id": 1, "name": "Example Result"}
        ]
    }))
}
```

## Contributing Guidelines

If you're contributing to the Alith SDK:

1. **Fork the Repository**: Create your own fork of the repository
2. **Create a Feature Branch**: `git checkout -b feature/my-feature`
3. **Make Your Changes**: Implement your feature or bug fix
4. **Run Tests**: Ensure all tests pass with `make test`
5. **Format Code**: Run `make fmt` and `make clippy` to ensure code quality
6. **Submit a Pull Request**: With a detailed description of your changes

## Related Documentation

* [Features/Tools.md](../Features/Tools.md) - Learn about the tool system in Alith
* [Features/Memory.md](../Features/Memory.md) - Memory implementations for Rust
* [Advanced/Web3Integration.md](../Advanced/Web3Integration.md) - Web3 features in Rust
* [GetStarted.md](../GetStarted.md) - General getting started guide
* [Developing/NodeJs.md](NodeJs.md) - Compare with Node.js development

## API Reference

```rust
// Agent Creation
pub struct Agent {
    // Fields omitted for brevity
}

impl Agent {
    pub fn new() -> Self;
    pub fn name(self, name: &str) -> Self;
    pub fn model(self, model: &str) -> Self;
    pub fn api_key(self, api_key: &str) -> Self;
    pub fn base_url(self, base_url: &str) -> Self;
    pub fn preamble(self, preamble: &str) -> Self;
    pub fn add_tool<T: Tool + 'static>(self, tool: T) -> Self;
    pub fn memory<M: Memory + 'static>(self, memory: M) -> Self;
    pub fn prompt(&self, input: &str) -> String;
    pub async fn prompt_async(&self, input: &str) -> Result<String, Error>;
}

// Tool Trait
pub trait Tool: Send + Sync {
    fn name(&self) -> &str;
    fn description(&self) -> &str;
    fn schema(&self) -> &str;
    fn execute(&self, args: Value, context: &ToolContext) -> ToolResult;
}
```

---

*Last Updated: 2025-03-16*
