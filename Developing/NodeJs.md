# Developing with Node.js

## Overview

This guide provides instructions for setting up a development environment for the Alith SDK using Node.js and TypeScript. The Node.js SDK offers a familiar JavaScript-based interface to Alith with full TypeScript typings, making it ideal for web applications, serverless functions, and integration with existing JavaScript ecosystems.

> **🟢 Node.js Development**: The Node.js SDK provides first-class TypeScript support and seamless integration with npm packages and modern JavaScript frameworks.

## Prerequisites

Before you begin developing with the Node.js SDK, ensure you have the following:

* **Node.js**: Version 18.0.0 or later
* **npm**: Node package manager (comes with Node.js)
* **TypeScript**: Version 5.0.0 or later (optional but recommended)
* **Git**: For version control
* **Optional**: An IDE with TypeScript support (VS Code, WebStorm, etc.)

## Setup Instructions

### Installing Node.js

If you don't have Node.js installed, download and install it from [nodejs.org](https://nodejs.org/). We recommend using the LTS version for development.

After installation, verify that Node.js and npm are properly installed:

```bash
node --version
# Expected output: v18.x.x or higher

npm --version
# Expected output: 8.x.x or higher
```

### Setting Up the Project

#### Option 1: Contributing to Alith SDK

If you're planning to contribute to the Alith Node.js SDK:

```bash
# Clone the repository
git clone https://github.com/0xLazAI/alith.git
cd alith

# Install dependencies
npm install

# Build the project
npm run build

# Run tests
npm test
```

#### Option 2: Starting a New Project with Alith

To create a new Node.js project that uses Alith:

```bash
# Create a new project directory
mkdir my-alith-project
cd my-alith-project

# Initialize package.json
npm init -y

# Install Alith
npm install alith

# Install TypeScript and ts-node for development
npm install --save-dev typescript ts-node @types/node

# Initialize TypeScript configuration
npx tsc --init

# Create a basic example
cat > index.ts << EOL
import { Agent, Tool } from 'alith';

// Define a simple tool
const weatherTool = new Tool({
  name: 'get_weather',
  description: 'Get the current weather for a location',
  schema: {
    type: 'object',
    properties: {
      location: {
        type: 'string',
        description: 'The city and state/country'
      }
    },
    required: ['location']
  },
  function: async (args: { location: string }) => {
    const { location } = args;
    return `The weather in ${location} is sunny and 72°F`;
  }
});

// Create an agent with the tool
const agent = new Agent({
  name: 'WeatherAssistant',
  model: 'gpt-4',
  apiKey: process.env.OPENAI_API_KEY,
  preamble: 'You are a helpful assistant that provides weather information.',
  tools: [weatherTool]
});

// Function to run the agent
async function main() {
  try {
    const response = await agent.prompt('What\'s the weather like in San Francisco?');
    console.log('Agent response:', response);
  } catch (error) {
    console.error('Error:', error);
  }
}

// Run the function
main();
EOL

# Create a script to run the example
echo '{
  "scripts": {
    "start": "ts-node index.ts"
  }
}' > package.json

# Run the example
npm start
```

## Development Workflow

### Managing Dependencies

For a new project, manage your dependencies with npm:

```bash
# Add Alith as a dependency
npm install alith

# Add development dependencies
npm install --save-dev typescript ts-node @types/node jest
```

### Building TypeScript Code

Set up a TypeScript build process:

```bash
# Create tsconfig.json
echo '{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "esModuleInterop": true,
    "strict": true,
    "outDir": "./dist",
    "declaration": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "**/*.test.ts"]
}' > tsconfig.json

# Add build script to package.json
echo '{
  "scripts": {
    "build": "tsc",
    "start": "node dist/index.js",
    "dev": "ts-node src/index.ts"
  }
}' > package.json

# Build the project
npm run build
```

### Running Tests

Set up and run tests with Jest:

```bash
# Install Jest
npm install --save-dev jest ts-jest @types/jest

# Create Jest config
echo 'module.exports = {
  preset: "ts-jest",
  testEnvironment: "node",
  testMatch: ["**/*.test.ts"]
};' > jest.config.js

# Add test script to package.json
npm pkg set scripts.test="jest"

# Run tests
npm test
```

## Advanced Usage

### Creating an Express API with Alith

Integrate Alith into an Express.js web application:

```typescript
import express from 'express';
import { Agent, Tool } from 'alith';
import dotenv from 'dotenv';

// Load environment variables
dotenv.config();

const app = express();
app.use(express.json());

// Create a database search tool
const searchTool = new Tool({
  name: 'search_database',
  description: 'Search the database for information',
  schema: {
    type: 'object',
    properties: {
      query: {
        type: 'string',
        description: 'The search query'
      }
    },
    required: ['query']
  },
  function: async (args: { query: string }) => {
    // In a real app, this would query a database
    return `Results for: ${args.query}`;
  }
});

// Create the agent
const agent = new Agent({
  name: 'APIAssistant',
  model: 'gpt-4',
  apiKey: process.env.OPENAI_API_KEY,
  preamble: 'You are an API assistant that helps users find information.',
  tools: [searchTool]
});

// Define the API endpoint
app.post('/api/ask', async (req, res) => {
  const { question } = req.body;
  
  if (!question) {
    return res.status(400).json({ error: 'No question provided' });
  }
  
  try {
    const response = await agent.prompt(question);
    return res.json({ answer: response });
  } catch (error) {
    console.error('Error:', error);
    return res.status(500).json({ error: 'An error occurred while processing your request' });
  }
});

// Start the server
const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`Server is running on port ${PORT}`);
});
```

### Using Memory with Agents

Implement conversation memory for stateful interactions:

```typescript
import { Agent, Tool, WindowBufferMemory } from 'alith';

// Create a memory instance
const memory = new WindowBufferMemory({
  capacity: 5 // Remember the last 5 exchanges
});

// Create an agent with memory
const agent = new Agent({
  name: 'MemoryAssistant',
  model: 'gpt-4',
  apiKey: process.env.OPENAI_API_KEY,
  preamble: 'You are a helpful assistant that remembers previous conversations.',
  memory
});

// Function to have a conversation
async function conversation() {
  try {
    // First message
    let response = await agent.prompt('My name is Alice');
    console.log('Assistant:', response);
    
    // Second message - the agent should remember the name
    response = await agent.prompt('What\'s my name?');
    console.log('Assistant:', response);
    
    // Third message - asking about a preference
    response = await agent.prompt('I love chocolate ice cream');
    console.log('Assistant:', response);
    
    // Fourth message - the agent should remember the preference
    response = await agent.prompt('What\'s my favorite ice cream?');
    console.log('Assistant:', response);
  } catch (error) {
    console.error('Error:', error);
  }
}

// Run the conversation
conversation();
```

## Best Practices

* **Environment Variables**: Use `dotenv` to manage API keys and configuration
* **TypeScript**: Leverage TypeScript for type safety and better IDE support
* **Error Handling**: Implement proper error handling for API calls and tool execution
* **Async/Await**: Use async/await pattern for all asynchronous operations
* **Modules**: Structure your code in modular, reusable components

> **💡 Tip:** When developing with TypeScript, take advantage of Alith's type definitions for better autocompletion and type checking. This will help catch errors before runtime.

## Common Issues and Solutions

### Issue 1: API Authentication

**Problem**: Issues with API key authentication.

**Solution**: Use environment variables with dotenv:

```typescript
// Install dotenv
// npm install dotenv

import dotenv from 'dotenv';
import { Agent } from 'alith';

// Load environment variables from .env file
dotenv.config();

// Create agent with API key from environment
const agent = new Agent({
  name: 'SecureAgent',
  model: 'gpt-4',
  apiKey: process.env.OPENAI_API_KEY
});

// Check if API key is available
if (!process.env.OPENAI_API_KEY) {
  console.error('Error: OPENAI_API_KEY not found in environment variables');
  process.exit(1);
}
```

### Issue 2: TypeScript Configuration

**Problem**: TypeScript compilation errors with the Alith SDK.

**Solution**: Ensure proper TypeScript configuration:

```json
// tsconfig.json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "esModuleInterop": true,
    "forceConsistentCasingInFileNames": true,
    "strict": true,
    "skipLibCheck": true,
    "outDir": "./dist",
    "declaration": true,
    "resolveJsonModule": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "**/*.test.ts", "dist"]
}
```

## Contributing Guidelines

If you're contributing to the Alith Node.js SDK:

1. **Fork the Repository**: Create your own fork of the repository
2. **Create a Feature Branch**: `git checkout -b feature/my-feature`
3. **Make Your Changes**: Implement your feature or bug fix
4. **Run Tests**: Ensure all tests pass with `npm test`
5. **Lint Your Code**: Run `npm run lint` to ensure code quality
6. **Submit a Pull Request**: With a detailed description of your changes

## Related Documentation

* [Features/Tools.md](../Features/Tools.md) - Learn about the tool system in Alith
* [Features/Memory.md](../Features/Memory.md) - Memory implementations for Node.js
* [Advanced/Web3Integration.md](../Advanced/Web3Integration.md) - Web3 features in Node.js
* [GetStarted.md](../GetStarted.md) - General getting started guide
* [Developing/Rust.md](Rust.md) - Compare with Rust development

## API Reference

```typescript
// Agent Creation
interface AgentConfig {
  name?: string;
  model: string;
  apiKey: string;
  baseUrl?: string;
  preamble?: string;
  tools?: Tool[];
  memory?: Memory;
}

class Agent {
  constructor(config: AgentConfig);
  
  async prompt(message: string): Promise<string>;
  
  addTool(tool: Tool): void;
  
  setMemory(memory: Memory): void;
}

// Tool Creation
interface ToolConfig {
  name: string;
  description: string;
  schema: object;
  function: (args: any) => Promise<any> | any;
}

class Tool {
  constructor(config: ToolConfig);
}

// Memory Creation
interface MemoryConfig {
  capacity?: number;
}

class WindowBufferMemory {
  constructor(config?: MemoryConfig);
  
  addUserMessage(message: string): void;
  
  addAIMessage(message: string): void;
  
  getMessages(): Message[];
  
  clear(): void;
}
```

---

*Last Updated: 2025-03-16*