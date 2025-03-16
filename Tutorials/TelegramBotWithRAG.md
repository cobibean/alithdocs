# Telegram Bot with RAG (Retrieval-Augmented Generation)

In this tutorial, you will learn how to create a Telegram Bot that uses the Alith Python SDK to generate responses to messages with the GitHub project documents. This bot will listen to messages in a Telegram channel and reply with contextually relevant information from your codebase.

> **See Also:** For a simpler Telegram Bot implementation without RAG, see the [Basic Telegram Bot Tutorial](TelegramBot.md).

Note: Although we used Python in this tutorial, you can still use the [Alith Rust SDK](../Developing/Rust.md) and [Node.js SDK](../Developing/NodeJs.md) to implement similar functionality. The advantage of using the Alith Python SDK is that it improves development efficiency while still providing a production-level AI Bot. For example, you can deploy the Bot on AWS Lambda, leveraging the core Rust implementation and minimal Python dependencies, resulting in a much smaller cold start time compared to frameworks like Langchain.

## Prerequisites

Before starting, ensure you have the following:

- Python 3.8 or higher
- A Telegram account
- OpenAI API key (or another supported AI model provider)
- Basic understanding of [Alith Tools](../Features/Tools.md) and [Knowledge integration](../Features/Knowledge.md)
- Familiarity with [RAG concepts](../Advanced/Retrieval-Augmented%20Generation%20(RAG).md)

## Step 1: Install Required Libraries

Install the necessary Python libraries using pip:

```bash
# For AI Agent with LLM
python3 -m pip install alith

# For Telegram Bot
python3 -m pip install python-telegram-bot

# For GitHub project document download and extract
python3 -m pip install langchain-community langchain_text_splitters

# For Vector Database and Embedding Models
python3 -m pip install -U pymilvus "pymilvus[model]"
```

## Step 2: Create a Telegram Bot

1. Search for BotFather in Telegram and interact with it.
2. Send the `/newbot` command to create a new bot.
3. Follow the prompts to provide a name and username for your bot, and save the generated Bot Token.

## Step 3: Set Up Environment Variables

Store your API keys and tokens as environment variables for security:

```bash
export GITHUB_ACCESS_KEY="your-github-access-key"
export OPENAI_API_KEY="your-openai-api-key"
export TELEGRAM_BOT_TOKEN="your-telegram-bot-token"
```

## Step 4: Write the Telegram Bot Code

Create a Python script (e.g., `tg-bot-with-rag.py`) and add the following code:

```python
import os
import re
 
from telegram import Update
from telegram.ext import (
    Application,
    MessageHandler,
    filters,
    CallbackContext,
)
 
from langchain_community.document_loaders.github import GithubFileLoader
from langchain_text_splitters import MarkdownTextSplitter
from pymilvus import MilvusClient, model
from alith import Agent
 
# --------------------------------------------
# Constants
# --------------------------------------------
 
GITHUB_ACCESS_KEY = os.getenv("GITHUB_ACCESS_KEY")
TELEGRAM_BOT_TOKEN = os.getenv("TELEGRAM_BOT_TOKEN")
GITHUB_REPO = "0xLazAI/alith"  # Replace with your repository
DOC_RELATIVE_PATH = "website/src/content"  # Path to documentation in repo
 
# --------------------------------------------
# Init Embeddings Model and Document Database
# --------------------------------------------
 
client = MilvusClient("alith.db")
client.create_collection(
    collection_name="alith",
    dimension=768,
)
# If connection to https://huggingface.co/ fails, uncomment the following path.
# os.environ["HF_ENDPOINT"] = "https://hf-mirror.com"
# Note: This will download a small embedding model "paraphrase-albert-small-v2" (~50MB) from Hugging Face.
embedding_fn = model.DefaultEmbeddingFunction()
 
 
def create_vector_store():
    """
    Load documents from GitHub repository, split them into chunks,
    and store them in the vector database.
    """
    docs = GithubFileLoader(
        repo=GITHUB_REPO,
        access_token=GITHUB_ACCESS_KEY,
        github_api_url="https://api.github.com",
        file_filter=lambda file_path: re.match(
            f"{DOC_RELATIVE_PATH}/.*\\.mdx?", file_path
        )
        is not None,
    ).load()
    text_splitter = MarkdownTextSplitter(chunk_size=2000, chunk_overlap=200)
    docs = [split.page_content for split in text_splitter.split_documents(docs)]
    vectors = embedding_fn.encode_documents(docs)
    data = [
        {"id": i, "vector": vectors[i], "text": docs[i], "subject": "documentation"}
        for i in range(len(vectors))
    ]
    client.insert(collection_name="alith", data=data)
 
 
# Initialize the vector store with documents
create_vector_store()
 
# --------------------------------------------
# Init Alith Agent
# --------------------------------------------
 
agent = Agent(
    name="Telegram Bot Agent",
    model="gpt-4",
    preamble="""You are a helpful assistant that answers questions about the Alith framework.
    Use the provided documentation to give accurate answers. If you don't know something,
    be honest about it rather than making up information.""",
)
 
 
def prompt_with_rag(text: str) -> str:
    """
    Generate a response using Retrieval-Augmented Generation.
    
    Args:
        text: The user's query text
        
    Returns:
        The generated response
    """
    # Convert the text query to vector
    query_vectors = embedding_fn.encode_queries([text])
    
    # Search from the vector database
    res = client.search(
        collection_name="alith",
        data=query_vectors,
        limit=2,  # Retrieve top 2 most relevant documents
        output_fields=["text", "subject"],
    )
    
    # Extract the retrieved documents
    docs = [d["entity"]["text"] for r in res for d in r]
    
    # Generate response with the agent using the retrieved documents as context
    response = agent.prompt(
        "{}\n\n<attachments>\n{}</attachments>\n".format(text, "\n".join(docs))
    )
    return response
 
 
# --------------------------------------------
# Init Telegram Bot
# --------------------------------------------
 
 
async def handle_message(update: Update, context: CallbackContext) -> None:
    """Handle incoming messages and generate responses."""
    # Get user's message and generate a response
    response = prompt_with_rag(update.message.text)
    # Send the response back to the user
    await context.bot.send_message(chat_id=update.effective_chat.id, text=response)
 
 
app = Application.builder().token(TELEGRAM_BOT_TOKEN).build()
app.add_handler(MessageHandler(filters.TEXT & (~filters.COMMAND), handle_message))
 
# Start the bot
if __name__ == "__main__":
    print("Starting Telegram bot...")
    app.run_polling()
```

## Step 5: Run the Telegram Bot

Run your Python script to start the bot:

```bash
python3 tg-bot-with-rag.py
```

## Step 6: Test the Bot

1. Open Telegram and search for your bot by its username
2. Start a conversation with your bot
3. Ask questions about the Alith framework, and the bot will respond using the documentation

Example:

**Input**: 
```
What is Alith?
```

**Output**:
```
Alith is a decentralized AI agent framework designed for Web3 and crypto applications. 
It provides a flexible, high-performance infrastructure for creating AI agents that can 
interact with blockchain data and various large language models (LLMs). 
Alith supports multiple programming languages including Python, Rust, and Node.js, 
making it versatile for different development needs.
```

## Step 7: Deployment Options

To keep the bot running 24/7, deploy it to a cloud platform:

- **AWS Lambda**: Use the Serverless Framework to deploy. Alith's minimal dependencies make it ideal for Lambda.
- **Heroku**: Follow the Heroku Python deployment guide.
- **Google Cloud Run**: Deploy as a containerized application.

## Customization Options

Here are some ways to enhance your bot:

1. **Conversation History**: Store conversation history to enable multi-turn dialogues.
2. **Custom Commands**: Add command handlers for specific functionality like `/help` or `/search`.
3. **Multiple Data Sources**: Integrate with other repositories or databases for more comprehensive knowledge.
4. **User Authentication**: Add authentication to restrict access to authorized users.
5. **Custom Agent Models**: Experiment with different LLM models or create specialized agents for different topics.

## Adding RAG Capabilities

In this section, we'll add document retrieval capabilities to our bot using Alith's [Knowledge feature](../Features/Knowledge.md).

## Testing and Deployment

## Extending the Bot

Here are some ideas to extend your RAG-enabled Telegram Bot:

1. Add support for multiple document sources
2. Implement different [embedding models](../Features/Embeddings.md) for better retrieval
3. Add [conversation memory](../Features/Memory.md) to maintain context
4. Implement user feedback mechanisms to improve responses

## Related Documentation

* [Basic Telegram Bot Tutorial](TelegramBot.md) - A simpler implementation without RAG
* [Knowledge Features](../Features/Knowledge.md) - Learn more about Alith's knowledge integration
* [RAG Advanced Guide](../Advanced/Retrieval-Augmented%20Generation%20(RAG).md) - In-depth coverage of RAG techniques
* [Embeddings](../Features/Embeddings.md) - Understanding vector embeddings in Alith
* [Python Development](../Developing/Python.md) - More information on developing with the Python SDK