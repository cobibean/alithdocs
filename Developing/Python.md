# Developing with Python

## Overview

This guide provides instructions for setting up a development environment for the Alith SDK using Python. Python offers a simple, intuitive interface to the Alith framework, making it ideal for rapid prototyping, data science applications, and integrating AI agents into existing Python workflows.

> **🐍 Python Development**: The Python SDK provides an intuitive, Pythonic interface to all Alith features, making it easy to create powerful AI agents with minimal boilerplate code.

## Prerequisites

Before you begin developing with the Python SDK, ensure you have the following:

* **Python**: Version 3.8 or later
* **pip**: Python package manager
* **Virtual Environment**: `venv` or `conda` (recommended)
* **Git**: For version control
* **Optional**: An IDE with Python support (VS Code, PyCharm, etc.)

## Setup Instructions

### Installing Python

If you don't have Python installed, download and install it from [python.org](https://python.org/downloads/) or use your operating system's package manager.

After installation, verify Python is properly installed:

```bash
python --version
# Expected output: Python 3.x.x
```

### Setting Up the Project

#### Option 1: Contributing to Alith SDK

If you're planning to contribute to the Alith Python SDK:

```bash
# Clone the repository
git clone https://github.com/0xLazAI/alith.git
cd alith

# Set up a virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install development dependencies
pip install -e ".[dev]"

# Run tests
pytest
```

#### Option 2: Starting a New Project with Alith

To create a new Python project that uses Alith:

```bash
# Create a project directory
mkdir my_alith_project
cd my_alith_project

# Set up a virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install Alith
pip install alith

# Create a basic example
cat > main.py << EOL
from alith import Agent, Tool
import os

# Define a simple tool
def get_weather(location):
    return f"The weather in {location} is sunny and 72°F"

weather_tool = Tool(
    name="get_weather",
    description="Get the current weather for a location",
    function=get_weather,
    schema={
        "type": "object",
        "properties": {
            "location": {
                "type": "string",
                "description": "The city and state/country"
            }
        },
        "required": ["location"]
    }
)

# Create an agent with the tool
agent = Agent(
    name="WeatherAssistant",
    model="gpt-4",
    api_key=os.environ.get("OPENAI_API_KEY"),
    preamble="You are a helpful assistant that provides weather information.",
    tools=[weather_tool]
)

# Prompt the agent
response = agent.prompt("What's the weather like in San Francisco?")
print(f"Agent response: {response}")
EOL

# Run the example
python main.py
```

## Development Workflow

### Managing Dependencies

For a new project, create a `requirements.txt` file:

```bash
echo "alith>=0.1.0" > requirements.txt
```

For contributing to the SDK, use the development extras:

```bash
pip install -e ".[dev,test,docs]"
```

### Running Tests

Run the test suite to ensure everything is working correctly:

```bash
# For projects using Alith
pytest

# For the Alith SDK itself
python -m pytest tests/
```

### Code Formatting and Linting

Ensure your code follows Python's formatting standards:

```bash
# Install formatting and linting tools
pip install black flake8 isort

# Format code
black .
isort .

# Run linter
flake8
```

## Advanced Usage

### Using Asynchronous Agents

Create agents that work asynchronously for better performance in web applications:

```python
import asyncio
from alith import Agent, Tool

async def main():
    # Create an async agent
    agent = Agent(
        name="AsyncAssistant", 
        model="gpt-4",
        api_key="your-api-key"
    )
    
    # Use async prompt method
    response = await agent.aprompt("Tell me about artificial intelligence")
    print(response)
    
    # Multiple concurrent prompts
    tasks = [
        agent.aprompt("What is Python?"),
        agent.aprompt("What is machine learning?"),
        agent.aprompt("What is natural language processing?")
    ]
    
    results = await asyncio.gather(*tasks)
    for i, result in enumerate(results):
        print(f"Answer {i+1}: {result}")

# Run the async code
asyncio.run(main())
```

### Creating a Flask API with Alith

Integrate Alith into a Flask web application:

```python
from flask import Flask, request, jsonify
from alith import Agent, Tool
import os

app = Flask(__name__)

# Create tools
def search_database(query):
    # In a real app, this would query a database
    return f"Results for: {query}"

search_tool = Tool(
    name="search_database",
    description="Search the database for information",
    function=search_database,
    schema={
        "type": "object",
        "properties": {
            "query": {
                "type": "string",
                "description": "The search query"
            }
        },
        "required": ["query"]
    }
)

# Create agent
agent = Agent(
    name="APIAssistant",
    model="gpt-4",
    api_key=os.environ.get("OPENAI_API_KEY"),
    preamble="You are an API assistant that helps users find information.",
    tools=[search_tool]
)

@app.route('/ask', methods=['POST'])
def ask():
    data = request.json
    question = data.get('question', '')
    
    if not question:
        return jsonify({"error": "No question provided"}), 400
    
    try:
        response = agent.prompt(question)
        return jsonify({"answer": response})
    except Exception as e:
        return jsonify({"error": str(e)}), 500

if __name__ == '__main__':
    app.run(debug=True)
```

## Best Practices

* **Environment Variables**: Store API keys in environment variables or use a `.env` file with python-dotenv
* **Error Handling**: Implement proper error handling for API calls and tool execution
* **Caching**: Use caching for repeated API calls to reduce costs and latency
* **Type Hints**: Add type hints to improve code readability and catch errors early
* **Documentation**: Document your tools and agents with clear docstrings

> **💡 Tip:** When creating an application with multiple agents, consider using a factory pattern to create and configure agents with different capabilities and personalities.

## Common Issues and Solutions

### Issue 1: API Key Management

**Problem**: Insecure storage of API keys.

**Solution**: Use environment variables and python-dotenv:

```python
# Install python-dotenv
# pip install python-dotenv

from dotenv import load_dotenv
import os

# Load environment variables from .env file
load_dotenv()

# Create agent with API key from environment
agent = Agent(
    name="SecureAgent",
    model="gpt-4",
    api_key=os.environ.get("OPENAI_API_KEY")
)
```

### Issue 2: Tool Argument Validation

**Problem**: Invalid arguments being passed to tools.

**Solution**: Add validation in your tool functions:

```python
def search_database(query):
    if not query or not isinstance(query, str):
        return "Error: Invalid query. Please provide a valid search term."
    
    # Proceed with valid query
    return f"Results for: {query}"
```

## Contributing Guidelines

If you're contributing to the Alith Python SDK:

1. **Fork the Repository**: Create your own fork of the repository
2. **Create a Feature Branch**: `git checkout -b feature/my-feature`
3. **Make Your Changes**: Implement your feature or bug fix
4. **Run Tests**: Ensure all tests pass with `pytest`
5. **Format Code**: Use Black and isort to ensure code quality
6. **Submit a Pull Request**: With a detailed description of your changes

## Related Documentation

* [Features/Tools.md](../Features/Tools.md) - Learn about the tool system in Alith
* [Features/Memory.md](../Features/Memory.md) - Memory implementations for Python
* [Advanced/Retrieval-Augmented Generation (RAG).md](../Advanced/Retrieval-Augmented%20Generation%20(RAG).md) - Building RAG systems with Python
* [GetStarted.md](../GetStarted.md) - General getting started guide
* [Developing/NodeJs.md](NodeJs.md) - Compare with Node.js development

## API Reference

```python
# Agent Creation
class Agent:
    def __init__(
        self,
        name: str = None,
        model: str = None,
        api_key: str = None,
        base_url: str = None,
        preamble: str = None,
        tools: List[Tool] = None,
        memory: Memory = None
    ):
        """Initialize an Alith agent."""
    
    def prompt(self, message: str) -> str:
        """Send a prompt to the agent and get a response."""
    
    async def aprompt(self, message: str) -> str:
        """Asynchronously send a prompt to the agent."""
    
    def add_tool(self, tool: Tool) -> None:
        """Add a tool to the agent."""

# Tool Creation
class Tool:
    def __init__(
        self,
        name: str,
        description: str,
        function: Callable,
        schema: Dict
    ):
        """Initialize a tool for the agent."""
```

---

*Last Updated: 2025-03-16*