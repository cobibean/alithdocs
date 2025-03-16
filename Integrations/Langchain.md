# LangChain Integration

## Overview

The LangChain integration enables you to use Alith as a powerful LLM component within LangChain workflows. This integration combines Alith's high-performance agent capabilities with LangChain's flexible composition framework, allowing you to build sophisticated AI applications that leverage the strengths of both ecosystems. Use Alith agents as LLM nodes in your existing LangChain workflows to enhance performance and capabilities.

> **🔗 Integration Feature**: The LangChain integration brings Alith's performance optimizations, tool support, and Web3 capabilities to your LangChain workflows, enabling more powerful and efficient applications.

## Key Concepts

* **AlithLLM**: A LangChain-compatible wrapper for Alith agents
* **Chainable Components**: Use Alith agents within LangChain's composable chains
* **Performance Optimization**: Leverage Alith's optimized inference for better LangChain performance
* **Tool Interoperability**: Use Alith tools within LangChain or LangChain tools within Alith
* **Cross-Framework Compatibility**: Combine the best features of both frameworks

## Implementation

### Basic Integration: Alith as LLM in LangChain

The most straightforward way to use Alith with LangChain is to wrap an Alith agent as a LangChain LLM:

```python
from langchain_core.prompts import PromptTemplate
from langchain_core.runnables import RunnablePassthrough
from langchain_core.output_parsers import StrOutputParser
from langchain_alith import AlithLLM
from alith import Agent
 
# Create an Alith agent
agent = Agent(
    name="QAAgent",
    model="deepseek-chat",  # or other models like "gpt-4", "claude-3", etc.
    api_key="your-api-key",  # Replace with your API key or read from environment
    base_url="api.deepseek.com",
    preamble="""As an adaptable question-answering assistant, your role is to leverage the provided context
    to address user inquiries. When direct answers are not apparent from the context, you are encouraged to
    draw upon analogies or related knowledge to formulate or infer solutions. If a certain answer remains
    elusive, politely acknowledge the limitation. Aim for concise responses, ideally within three sentences."""
)
 
# Create a LangChain prompt template
prompt = PromptTemplate.from_template(
    """Answer the following question based on the provided context.
 
Question: {question}
"""
)
 
# Wrap the Alith agent as a LangChain LLM
llm = AlithLLM(agent=agent)

# Create a simple chain
qa_chain = {"question": RunnablePassthrough()} | prompt | llm | StrOutputParser()

# Invoke the chain
response = qa_chain.invoke("What is the capital of France?")
print(response)
```

### Advanced Integration: RAG with Alith and LangChain

Here's a more comprehensive example that implements a Retrieval-Augmented Generation (RAG) system:

```python
from langchain_core.prompts import PromptTemplate
from langchain_core.runnables import RunnablePassthrough
from langchain_core.output_parsers import StrOutputParser
from langchain.vectorstores import Chroma
from langchain.embeddings import OpenAIEmbeddings
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain_alith import AlithLLM
from alith import Agent, Tool
import os

# Create a custom tool for the Alith agent
web_search_tool = Tool(
    name="web_search",
    description="Search for information on the web",
    function=lambda query: f"Search results for: {query}",
    schema={
        "type": "object",
        "properties": {
            "query": {"type": "string", "description": "The search query"}
        },
        "required": ["query"]
    }
)

# Create an Alith agent with tools
agent = Agent(
    name="RAGAgent",
    model="gpt-4",
    api_key=os.environ.get("OPENAI_API_KEY"),
    preamble="""You are a research assistant who provides accurate information based on
    the provided context and your knowledge. Use the tools available when needed.""",
    tools=[web_search_tool]
)

# Load documents and create vector store
documents = [
    "Paris is the capital of France.",
    "Berlin is the capital of Germany.",
    "Rome is the capital of Italy.",
    "Madrid is the capital of Spain."
]

text_splitter = RecursiveCharacterTextSplitter(chunk_size=1000, chunk_overlap=100)
splits = text_splitter.create_documents(documents)

embeddings = OpenAIEmbeddings()
vectorstore = Chroma.from_documents(documents=splits, embedding=embeddings)
retriever = vectorstore.as_retriever()

# Create the LangChain prompt
template = """Answer the question based on the following context:

Context: {context}

Question: {question}

If the answer is not in the context, use your tools to find the information.
"""

prompt = PromptTemplate.from_template(template)

# Wrap Alith agent as LangChain LLM
llm = AlithLLM(agent=agent)

# Define a function to format the context
def format_context(result):
    return "\n".join([doc.page_content for doc in result])

# Build the RAG chain
rag_chain = (
    {
        "context": retriever | format_context,
        "question": RunnablePassthrough()
    }
    | prompt
    | llm
    | StrOutputParser()
)

# Run the chain
response = rag_chain.invoke("What is the capital of France?")
print(response)
```

### Using LangChain Tools in Alith

You can also use LangChain tools within an Alith agent:

```python
from alith import Agent
from alith.integrations import LangChainToolAdapter
from langchain.tools import Tool as LCTool
from langchain.utilities import SerpAPIWrapper

# Create a LangChain tool
search = SerpAPIWrapper()
langchain_search_tool = LCTool(
    name="web_search",
    description="Search the web for information",
    func=search.run
)

# Adapt the LangChain tool for Alith
alith_search_tool = LangChainToolAdapter(langchain_search_tool)

# Create an Alith agent with the adapted LangChain tool
agent = Agent(
    name="SearchAgent",
    model="gpt-4",
    preamble="You are a research assistant who can search the web for information.",
    tools=[alith_search_tool]
)

# Use the agent with the LangChain tool
response = agent.prompt("What are the latest developments in AI safety?")
print(response)
```

## Configuration Options

| Option | Type | Description |
|--------|------|-------------|
| `agent` | Agent | The Alith agent to wrap as a LangChain LLM |
| `streaming` | boolean | Whether to enable streaming responses |
| `verbose` | boolean | Enable verbose logging for debugging |
| `return_exceptions` | boolean | Whether to return exceptions instead of raising them |
| `timeout` | number | Timeout in seconds for LLM calls |
| `max_retries` | number | Maximum number of retries for failed calls |
| `callbacks` | list | LangChain callbacks for monitoring and logging |

## Common Patterns

### Pattern 1: Chain Routing with Alith

Use Alith's decision-making capabilities to route between different LangChain chains:

```python
from alith import Agent
from langchain_alith import AlithLLM, AlithChainRouter
from langchain_core.prompts import PromptTemplate
from langchain_core.output_parsers import StrOutputParser

# Create specialized chains
math_chain = PromptTemplate.from_template("Solve this math problem: {question}") | math_llm | StrOutputParser()
history_chain = PromptTemplate.from_template("Answer this history question: {question}") | history_llm | StrOutputParser()
science_chain = PromptTemplate.from_template("Explain this scientific concept: {question}") | science_llm | StrOutputParser()

# Create an Alith agent for routing
router_agent = Agent(
    name="RouterAgent",
    model="gpt-3.5-turbo",  # Faster model for routing
    preamble="You classify user questions into categories: math, history, or science."
)

# Create a chain router using the Alith agent
router = AlithChainRouter(
    agent=router_agent,
    chains={
        "math": math_chain,
        "history": history_chain,
        "science": science_chain
    }
)

# Route questions to the appropriate chain
response = router.route("What is the square root of 144?")  # Routes to math_chain
print(response)
```

### Pattern 2: Hybrid Memory Management

Combine Alith's memory systems with LangChain's memory components:

```python
from alith import Agent, WindowBufferMemory
from langchain_alith import AlithLLM, AlithMemoryAdapter
from langchain.memory import ConversationBufferMemory
from langchain.chains import ConversationChain

# Create Alith agent with memory
alith_memory = WindowBufferMemory(capacity=5)
agent = Agent(
    name="ConversationalAgent",
    model="gpt-4",
    preamble="You are a helpful assistant with memory of previous conversations.",
    memory=alith_memory
)

# Adapt Alith memory for LangChain
langchain_memory = AlithMemoryAdapter(alith_memory)

# Create LangChain conversation chain
llm = AlithLLM(agent=agent)
conversation = ConversationChain(
    llm=llm,
    memory=langchain_memory,
    verbose=True
)

# Have a conversation
response1 = conversation.predict(input="My name is Alice")
print(response1)

response2 = conversation.predict(input="What's my name?")
print(response2)  # Should remember the name "Alice"
```

## Best Practices

* **Choose the Right Integration Level**: Decide whether to use Alith as a component in LangChain or vice versa based on your primary needs
* **Optimize for Performance**: Use Alith's performance optimizations for compute-intensive operations
* **Leverage Tool Ecosystems**: Combine tools from both frameworks for maximum flexibility
* **Monitor Token Usage**: Be aware of token consumption when chaining multiple components
* **Handle Errors Gracefully**: Implement proper error handling across framework boundaries

> **💡 Tip:** For complex, multi-step workflows with specialized components, use LangChain's composition capabilities. For high-performance agent behavior with Web3 integration, prioritize Alith's native features.

## Common Issues and Solutions

### Issue 1: Inconsistent Output Formats

Outputs from Alith might not match the expected format for LangChain components.

**Solution**: Use output parsers to ensure consistent formats:

```python
from langchain_core.output_parsers import PydanticOutputParser
from pydantic import BaseModel, Field

class SearchResult(BaseModel):
    relevance: float = Field(description="Relevance score from 0 to 1")
    answer: str = Field(description="The answer to the question")
    sources: list[str] = Field(description="List of source URLs")

parser = PydanticOutputParser(pydantic_object=SearchResult)

# Include the parser in the prompt
prompt = PromptTemplate.from_template(
    """Answer the question based on the context.
    
    {format_instructions}
    
    Question: {question}
    Context: {context}
    """
).partial(format_instructions=parser.get_format_instructions())

# Use in the chain
chain = prompt | AlithLLM(agent=agent) | parser
```

### Issue 2: Memory Synchronization

When using both frameworks' memory systems, they may get out of sync.

**Solution**: Implement a synchronization mechanism:

```python
class SynchronizedMemory:
    def __init__(self, alith_memory, langchain_memory):
        self.alith_memory = alith_memory
        self.langchain_memory = langchain_memory
        
    def add_user_message(self, message):
        # Add to both memories
        self.alith_memory.add_user_message(message)
        self.langchain_memory.chat_memory.add_user_message(message)
        
    def add_ai_message(self, message):
        # Add to both memories
        self.alith_memory.add_ai_message(message)
        self.langchain_memory.chat_memory.add_ai_message(message)
        
    def clear(self):
        # Clear both memories
        self.alith_memory.clear()
        self.langchain_memory.clear()
```

## Related Documentation

* [Eliza.md](Eliza.md) - Compare with alternative integration options
* [Features/Tools.md](../Features/Tools.md) - Learn about Alith tools that can be used with LangChain
* [Advanced/Retrieval-Augmented Generation (RAG).md](../Advanced/Retrieval-Augmented%20Generation%20(RAG).md) - Advanced RAG patterns
* [Tutorials/SearchBot.md](../Tutorials/SearchBot.md) - Build a search bot using Alith and LangChain
* [Developing/Python.md](../Developing/Python.md) - Python development environment setup

## API Reference

```python
# AlithLLM class for LangChain
class AlithLLM:
    def __init__(
        self, 
        agent: Agent, 
        streaming: bool = False,
        verbose: bool = False,
        callbacks: list = None,
        **kwargs
    ):
        """Initialize the AlithLLM for use in LangChain."""
        
    def invoke(self, prompt: str, **kwargs) -> str:
        """Invoke the Alith agent with a prompt and return the result."""
        
    def stream(self, prompt: str, **kwargs) -> Iterator[str]:
        """Stream the response from the Alith agent."""

# Tool Adapters
class LangChainToolAdapter:
    def __init__(self, langchain_tool, **kwargs):
        """Adapt a LangChain tool for use in Alith."""
        
class AlithToolAdapter:
    def __init__(self, alith_tool, **kwargs):
        """Adapt an Alith tool for use in LangChain."""
```

---

*Last Updated: 2025-03-16*
