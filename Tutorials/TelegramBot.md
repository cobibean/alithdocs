# Building a Telegram Bot with Alith

This tutorial guides you through building an intelligent Telegram bot powered by Alith. Your bot will be able to have natural conversations, respond to commands, and access on-chain data.

## What You'll Learn

- Setting up a Telegram bot with BotFather
- Integrating Alith agents with Telegram's API
- Adding conversation memory to your bot
- Implementing custom tools for Web3 functionality
- Deploying your bot to a server

## Prerequisites

- Node.js v16+ installed
- Basic JavaScript knowledge
- Telegram account
- OpenAI or Anthropic API key
- (Optional) Infura or Alchemy account for blockchain integration

## Step 1: Create a Telegram Bot

1. Open Telegram and search for [@BotFather](https://t.me/BotFather)
2. Send the command `/newbot`
3. Follow the prompts to name your bot
4. Save the API token provided by BotFather (it looks like `123456789:ABCDefGhIJKlmNoPQRsTUVwxyZ`)

## Step 2: Set Up Your Project

Create a new directory and initialize your project:

```bash
mkdir alith-telegram-bot
cd alith-telegram-bot
npm init -y
```

Install the required dependencies:

```bash
npm install alith telegraf dotenv axios
```

Create a `.env` file for your API keys:

```
TELEGRAM_BOT_TOKEN=your_telegram_token_here
OPENAI_API_KEY=your_openai_key_here
# Optional: Add blockchain provider keys
INFURA_API_KEY=your_infura_key_here
```

## Step 3: Create the Bot Structure

Create an `index.js` file with this basic structure:

```javascript
require('dotenv').config();
const { Agent } = require('alith');
const { Telegraf } = require('telegraf');

// Simple conversation history storage
const conversationHistory = new Map();
const MAX_HISTORY_LENGTH = 10; // Store last 10 messages per user

// Initialize your Alith Agent
const agent = new Agent({
  name: 'Telegram Bot Agent',
  model: 'gpt-4', // Or another model
  preamble: 'You are a helpful assistant in a Telegram chat. Provide concise, accurate responses.'
});

// Initialize Telegram Bot
const bot = new Telegraf(process.env.TELEGRAM_BOT_TOKEN);

// Handle /start command
bot.start((ctx) => {
  // Clear history when a new conversation starts
  const userId = ctx.from.id.toString();
  conversationHistory.set(userId, []);
  
  ctx.reply(
    "👋 Hello! I'm your AI assistant powered by Alith. How can I help you today?"
  );
});

// Handle /help command
bot.help((ctx) => {
  ctx.reply(
    `Here are commands I understand:
    
/start - Start or restart our conversation
/help - Show this help message
/clear - Clear our conversation history

You can also just chat with me normally!`
  );
});

// Add a command to clear conversation history
bot.command('clear', (ctx) => {
  const userId = ctx.from.id.toString();
  conversationHistory.set(userId, []);
  ctx.reply("Memory cleared! What would you like to talk about?");
});

// Handle messages
bot.on('text', async (ctx) => {
  try {
    const userId = ctx.from.id.toString();
    const userMessage = ctx.message.text;
    
    console.log(`Received message from ${userId}: ${userMessage}`);
    
    // Initialize conversation history for this user if it doesn't exist
    if (!conversationHistory.has(userId)) {
      conversationHistory.set(userId, []);
    }
    
    // Get current conversation history
    const history = conversationHistory.get(userId);
    
    // Add user message to history
    history.push({ role: 'user', content: userMessage });
    
    // Limit history size
    while (history.length > MAX_HISTORY_LENGTH) {
      history.shift();
    }
    
    // Create a prompt that includes conversation history
    let fullPrompt = '';
    
    if (history.length > 1) {
      // Add conversation history context if we have previous messages
      fullPrompt = "Here's our conversation so far:\n\n";
      
      for (const msg of history) {
        if (msg.role === 'user') {
          fullPrompt += `User: ${msg.content}\n`;
        } else {
          fullPrompt += `You: ${msg.content}\n`;
        }
      }
      
      // Add the current query
      fullPrompt += "\nBased on our conversation above, please respond to the user's most recent message.";
    } else {
      // If it's the first message, just use the message directly
      fullPrompt = userMessage;
    }
    
    console.log('Sending prompt with history to Alith agent');
    
    // Use the agent to generate a response
    const response = await agent.prompt(fullPrompt);
    console.log(`Generated response: ${response}`);
    
    // Add agent's response to history
    history.push({ role: 'assistant', content: response });
    
    // Send the reply back to the Telegram chat
    ctx.reply(response);
  } catch (error) {
    console.error('Error:', error);
    ctx.reply('Sorry, I encountered an error processing your request.');
  }
});

// Start the bot
bot.launch();

// Enable graceful stop
process.once('SIGINT', () => bot.stop('SIGINT'));
process.once('SIGTERM', () => bot.stop('SIGTERM'));

console.log('Bot is running...');
```

## Step 4: Add Custom Tools

Let's enhance our bot with a crypto price checking tool:

```javascript
// Add this after your imports
const axios = require('axios');

// Create price checking tool
const cryptoPriceTool = {
  name: 'check_crypto_price',
  description: 'Get the current price of a cryptocurrency',
  parameters: {
    type: 'object',
    properties: {
      symbol: {
        type: 'string',
        description: 'The cryptocurrency symbol (e.g., BTC, ETH)'
      }
    },
    required: ['symbol']
  },
  handler: async (params) => {
    try {
      const response = await axios.get(
        `https://api.coingecko.com/api/v3/simple/price?ids=${params.symbol.toLowerCase()}&vs_currencies=usd`
      );
      
      const data = response.data;
      const symbolLower = params.symbol.toLowerCase();
      
      if (data[symbolLower] && data[symbolLower].usd) {
        return `The current price of ${params.symbol.toUpperCase()} is $${data[symbolLower].usd}`;
      } else {
        return `Could not find price for ${params.symbol}`;
      }
    } catch (error) {
      console.error('API error:', error.message);
      return `Error fetching price: ${error.message}`;
    }
  }
};

// Update your agent initialization to include the tool
const agent = new Agent({
  name: 'Telegram Bot Agent',
  model: 'gpt-4',
  preamble: 'You are a helpful assistant in a Telegram chat. Provide concise, accurate responses. When asked about cryptocurrency prices, use the check_crypto_price tool.',
  tools: [cryptoPriceTool]
});
```

## Step 5: Add Blockchain Integration

Let's add Ethereum blockchain integration:

```javascript
// Add this after your imports
const { ethers } = require('ethers');

// Initialize Ethereum provider
const provider = new ethers.providers.JsonRpcProvider(
  `https://mainnet.infura.io/v3/${process.env.INFURA_API_KEY}`
);

// Create Ethereum balance checking tool
const ethBalanceTool = {
  name: 'check_eth_balance',
  description: 'Get the ETH balance of an Ethereum address or ENS name',
  parameters: {
    type: 'object',
    properties: {
      address: {
        type: 'string',
        description: 'Ethereum address or ENS name'
      }
    },
    required: ['address']
  },
  handler: async (params) => {
    try {
      const balance = await provider.getBalance(params.address);
      const ethBalance = ethers.utils.formatEther(balance);
      return `The ETH balance of ${params.address} is ${ethBalance} ETH`;
    } catch (error) {
      console.error('Blockchain error:', error.message);
      return `Error checking balance: ${error.message}`;
    }
  }
};

// Update your agent initialization to include both tools
const agent = new Agent({
  name: 'Telegram Bot Agent',
  model: 'gpt-4',
  preamble: 'You are a helpful assistant in a Telegram chat. Provide concise, accurate responses. You can check cryptocurrency prices and Ethereum balances.',
  tools: [cryptoPriceTool, ethBalanceTool]
});
```

## Step 6: Run and Test Your Bot

Run your bot locally:

```bash
node index.js
```

Open Telegram and search for your bot by its username. Test it with:

- Basic questions: "What is blockchain?"
- Price checks: "What's the current price of Ethereum?"
- Balance checks: "What's the ETH balance of vitalik.eth?"
- Memory tests: "What did we just talk about?"

## Step 7: Deploy to a Server

For production, deploy your bot to a server:

1. Set up a VPS (Digital Ocean, AWS, etc.)
2. Install Node.js and PM2
3. Copy your code and install dependencies
4. Create a proper .env file
5. Run with PM2 for persistence:

```bash
npm install -g pm2
pm2 start index.js --name "alith-telegram-bot"
pm2 save
pm2 startup
```

## Extending Your Bot

Here are some ideas to enhance your bot:

1. **Personality Customization**: Give your bot a unique character and voice
2. **NFT Analysis Tools**: Add tools to check NFT metadata and floor prices
3. **Multi-chain Support**: Expand to support multiple blockchains
4. **User Profiles**: Store user preferences and interests
5. **Scheduled Messages**: Send alerts for price movements or smart contract events

## Troubleshooting

Common issues and solutions:

1. **Bot not responding**: Check your Telegram token and ensure the bot is running
2. **Rate limiting**: Add retry logic and rate limiting for API requests
3. **Memory issues**: Implement a database for permanent storage of conversation history
4. **Tool execution errors**: Add better error handling and fallbacks

## Conclusion

You've built a powerful Telegram bot powered by Alith! It can have natural conversations, check cryptocurrency prices, and query Ethereum balances. This foundation can be extended for more complex use cases and integrations.

For a working example of a Telegram bot built with Alith, check out [Vic](https://github.com/cobibean/alith-telegram-bot), a sports prediction assistant with a unique personality.