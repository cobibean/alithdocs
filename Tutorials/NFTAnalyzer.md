# Building an NFT Analyzer with Alith

This tutorial guides you through creating an AI-powered NFT analyzer that can evaluate collections, track floor prices, analyze metadata, and generate insights about digital assets.

## What You'll Learn

- Fetching NFT data from blockchain and marketplaces
- Creating specialized NFT analysis tools
- Building an agent that understands NFT collections and traits
- Generating market and rarity insights
- Visualizing NFT data and trends

## Prerequisites

- Node.js v16+ installed
- Basic JavaScript knowledge
- OpenAI or Anthropic API key
- Ethereum node access (via Infura, Alchemy, etc.)
- OpenSea API key (optional but recommended)

## Step 1: Set Up Your Project

Create a new directory and initialize your project:

```bash
mkdir nft-analyzer
cd nft-analyzer
npm init -y
```

Install the required dependencies:

```bash
npm install alith dotenv ethers axios node-fetch@2 chart.js canvas
```

Create a `.env` file for your API keys:

```
OPENAI_API_KEY=your_openai_key_here
INFURA_API_KEY=your_infura_key_here
OPENSEA_API_KEY=your_opensea_key_here
ALCHEMY_API_KEY=your_alchemy_key_here
```

## Step 2: Create the Basic Analyzer Structure

Create a file named `analyzer.js` with the following structure:

```javascript
require('dotenv').config();
const { Agent } = require('alith');
const { ethers } = require('ethers');
const axios = require('axios');
const fetch = require('node-fetch');
const { createCanvas } = require('canvas');
const fs = require('fs');

// Initialize Ethereum provider (using Alchemy for better NFT support)
const provider = new ethers.providers.JsonRpcProvider(
  `https://eth-mainnet.alchemyapi.io/v2/${process.env.ALCHEMY_API_KEY}`
);

// Standard ERC721 interface
const ERC721_ABI = [
  "function name() view returns (string)",
  "function symbol() view returns (string)",
  "function totalSupply() view returns (uint256)",
  "function balanceOf(address owner) view returns (uint256)",
  "function ownerOf(uint256 tokenId) view returns (address)",
  "function tokenURI(uint256 tokenId) view returns (string)"
];

// Create a tool to check NFT ownership
const getNFTOwnershipTool = {
  name: 'get_nft_ownership',
  description: 'Check NFT ownership for a specific address',
  parameters: {
    type: 'object',
    properties: {
      address: {
        type: 'string',
        description: 'Ethereum address to check'
      },
      contract_address: {
        type: 'string',
        description: 'NFT contract address'
      }
    },
    required: ['address', 'contract_address']
  },
  handler: async (params) => {
    try {
      const { address, contract_address } = params;
      
      // Validate addresses
      if (!ethers.utils.isAddress(address) || !ethers.utils.isAddress(contract_address)) {
        return "Invalid Ethereum address provided";
      }
      
      // Create contract instance
      const nftContract = new ethers.Contract(contract_address, ERC721_ABI, provider);
      
      // Get collection info
      const name = await nftContract.name();
      const symbol = await nftContract.symbol();
      
      // Get balance
      const balance = await nftContract.balanceOf(address);
      const balanceNumber = balance.toNumber();
      
      if (balanceNumber === 0) {
        return `Address ${address} owns 0 NFTs from collection ${name} (${symbol})`;
      }
      
      // For collections with reasonable size, we can try to find which tokens they own
      // This is a simplified approach and not efficient for large collections
      let ownedTokens = [];
      
      try {
        // Try to get total supply (not all contracts implement this)
        const totalSupply = await nftContract.totalSupply();
        const supplyNumber = totalSupply.toNumber();
        
        // If supply is too large, we'll skip detailed ownership check
        if (supplyNumber > 1000) {
          return `Address ${address} owns ${balanceNumber} NFTs from collection ${name} (${symbol}). Collection is too large to enumerate owned tokens efficiently.`;
        }
        
        // For smaller collections, try to find owned tokens
        for (let i = 0; i < supplyNumber && ownedTokens.length < balanceNumber; i++) {
          try {
            const owner = await nftContract.ownerOf(i);
            if (owner.toLowerCase() === address.toLowerCase()) {
              ownedTokens.push(i);
            }
          } catch (err) {
            // Skip errors - some tokens might be burned or have non-sequential IDs
          }
        }
      } catch (err) {
        // totalSupply() might not be implemented
        return `Address ${address} owns ${balanceNumber} NFTs from collection ${name} (${symbol}). Cannot determine specific token IDs.`;
      }
      
      if (ownedTokens.length > 0) {
        return `Address ${address} owns ${balanceNumber} NFTs from collection ${name} (${symbol}). Token IDs: ${ownedTokens.join(', ')}`;
      } else {
        return `Address ${address} owns ${balanceNumber} NFTs from collection ${name} (${symbol}). Could not determine specific token IDs.`;
      }
    } catch (error) {
      return `Error checking NFT ownership: ${error.message}`;
    }
  }
};

// Create a tool to fetch NFT metadata
const getNFTMetadataTool = {
  name: 'get_nft_metadata',
  description: 'Get metadata for a specific NFT',
  parameters: {
    type: 'object',
    properties: {
      contract_address: {
        type: 'string',
        description: 'NFT contract address'
      },
      token_id: {
        type: 'integer',
        description: 'Token ID of the NFT'
      }
    },
    required: ['contract_address', 'token_id']
  },
  handler: async (params) => {
    try {
      const { contract_address, token_id } = params;
      
      // Validate address
      if (!ethers.utils.isAddress(contract_address)) {
        return "Invalid Ethereum contract address provided";
      }
      
      // Create contract instance
      const nftContract = new ethers.Contract(contract_address, ERC721_ABI, provider);
      
      // Get basic info
      const name = await nftContract.name();
      const symbol = await nftContract.symbol();
      
      // Get token URI
      let tokenURI;
      try {
        tokenURI = await nftContract.tokenURI(token_id);
      } catch (error) {
        return `Error retrieving tokenURI: ${error.message}. The token may not exist or the contract doesn't support tokenURI.`;
      }
      
      // Clean up IPFS URI if needed
      if (tokenURI.startsWith('ipfs://')) {
        tokenURI = tokenURI.replace('ipfs://', 'https://ipfs.io/ipfs/');
      }
      
      // Fetch metadata
      let metadata;
      try {
        const response = await axios.get(tokenURI);
        metadata = response.data;
      } catch (error) {
        return `Error fetching metadata from ${tokenURI}: ${error.message}`;
      }
      
      // Format the response
      let result = `NFT Metadata for ${name} (${symbol}) - Token ID: ${token_id}\n\n`;
      
      if (metadata.name) {
        result += `Name: ${metadata.name}\n`;
      }
      
      if (metadata.description) {
        result += `Description: ${metadata.description}\n`;
      }
      
      if (metadata.image) {
        let imageUrl = metadata.image;
        if (imageUrl.startsWith('ipfs://')) {
          imageUrl = imageUrl.replace('ipfs://', 'https://ipfs.io/ipfs/');
        }
        result += `Image: ${imageUrl}\n`;
      }
      
      if (metadata.attributes && metadata.attributes.length > 0) {
        result += '\nAttributes:\n';
        
        metadata.attributes.forEach(attr => {
          if (attr.trait_type && attr.value) {
            result += `- ${attr.trait_type}: ${attr.value}\n`;
          }
        });
      }
      
      return result;
    } catch (error) {
      return `Error retrieving NFT metadata: ${error.message}`;
    }
  }
};

// Initialize the Alith agent
const agent = new Agent({
  name: 'NFT Analyzer',
  model: 'gpt-4',
  preamble: `You are an NFT analysis assistant powered by AI. You help users understand 
  NFT collections, analyze traits, evaluate rarity, and provide insights about digital assets. 
  You can look up NFT metadata, check ownership, and analyze collections.
  
  When providing information:
  - Explain NFT concepts clearly for both beginners and experts
  - Provide objective analysis of collections and traits
  - Describe how rarity and traits may impact value
  - Clarify that your analysis is not financial advice`,
  tools: [
    getNFTOwnershipTool,
    getNFTMetadataTool
  ]
});

// Create a simple command-line interface
async function main() {
  console.log("NFT Analyzer powered by Alith");
  console.log("Type 'exit' to quit");
  
  const readline = require('readline').createInterface({
    input: process.stdin,
    output: process.stdout
  });
  
  const askQuestion = () => {
    readline.question("\nHow can I help with NFT analysis? ", async (query) => {
      if (query.toLowerCase() === 'exit') {
        console.log("Thank you for using the NFT Analyzer. Goodbye!");
        readline.close();
        return;
      }
      
      try {
        console.log("Analyzing...");
        const response = await agent.prompt(query);
        console.log("\nAnalyzer Response:");
        console.log(response);
        askQuestion();
      } catch (error) {
        console.error("Error:", error.message);
        askQuestion();
      }
    });
  };
  
  askQuestion();
}

main();
```

## Step 3: Add Collection Analysis Tool

Let's add a tool to analyze entire NFT collections:

```javascript
// Add this function after your imports
const getCollectionInfoTool = {
  name: 'get_collection_info',
  description: 'Get information about an NFT collection',
  parameters: {
    type: 'object',
    properties: {
      contract_address: {
        type: 'string',
        description: 'NFT contract address'
      }
    },
    required: ['contract_address']
  },
  handler: async (params) => {
    try {
      const { contract_address } = params;
      
      // Validate address
      if (!ethers.utils.isAddress(contract_address)) {
        return "Invalid Ethereum contract address provided";
      }
      
      // Create contract instance
      const nftContract = new ethers.Contract(contract_address, ERC721_ABI, provider);
      
      // Get basic collection info
      const name = await nftContract.name();
      const symbol = await nftContract.symbol();
      
      // Try to get total supply
      let totalSupply = "Unknown";
      try {
        const supply = await nftContract.totalSupply();
        totalSupply = supply.toString();
      } catch (err) {
        // totalSupply() might not be implemented
      }
      
      // Fetch additional data from OpenSea API if key is available
      let openseaData = {};
      if (process.env.OPENSEA_API_KEY) {
        try {
          const response = await fetch(
            `https://api.opensea.io/api/v1/asset_contract/${contract_address}`,
            {
              headers: {
                'X-API-KEY': process.env.OPENSEA_API_KEY
              }
            }
          );
          
          if (response.ok) {
            const data = await response.json();
            openseaData = {
              description: data.collection?.description || "No description available",
              imageUrl: data.collection?.image_url || null,
              externalUrl: data.collection?.external_url || null,
              discordUrl: data.collection?.discord_url || null,
              twitterUsername: data.collection?.twitter_username || null,
              floorPrice: data.collection?.stats?.floor_price || null,
              totalVolume: data.collection?.stats?.total_volume || null,
              numOwners: data.collection?.stats?.num_owners || null
            };
          }
        } catch (err) {
          console.log("Error fetching OpenSea data:", err.message);
        }
      }
      
      // Format the response
      let result = `NFT Collection Information: ${name} (${symbol})\n`;
      
      result += `Contract: ${contract_address}\n`;
      result += `Total Supply: ${totalSupply}\n\n`;
      
      if (openseaData.description) {
        result += `Description: ${openseaData.description}\n\n`;
      }
      
      if (openseaData.floorPrice) {
        result += `Floor Price: ${openseaData.floorPrice} ETH\n`;
      }
      
      if (openseaData.totalVolume) {
        result += `Total Volume: ${openseaData.totalVolume} ETH\n`;
      }
      
      if (openseaData.numOwners) {
        result += `Unique Owners: ${openseaData.numOwners}\n`;
      }
      
      if (openseaData.externalUrl) {
        result += `Website: ${openseaData.externalUrl}\n`;
      }
      
      if (openseaData.twitterUsername) {
        result += `Twitter: @${openseaData.twitterUsername}\n`;
      }
      
      if (openseaData.discordUrl) {
        result += `Discord: ${openseaData.discordUrl}\n`;
      }
      
      return result;
    } catch (error) {
      return `Error retrieving collection information: ${error.message}`;
    }
  }
};

// Add this tool to your agent's tools list
```

## Step 4: Add Trait Rarity Analysis

Let's add a tool to analyze trait rarity for a collection:

```javascript
const analyzeTraitRarityTool = {
  name: 'analyze_trait_rarity',
  description: 'Analyze trait rarity for an NFT',
  parameters: {
    type: 'object',
    properties: {
      contract_address: {
        type: 'string',
        description: 'NFT contract address'
      },
      token_id: {
        type: 'integer',
        description: 'Token ID of the NFT'
      }
    },
    required: ['contract_address', 'token_id']
  },
  handler: async (params) => {
    try {
      const { contract_address, token_id } = params;
      
      // Use Alchemy API for trait analysis if available
      if (!process.env.ALCHEMY_API_KEY) {
        return "This feature requires an Alchemy API key";
      }
      
      // Fetch NFT metadata from Alchemy
      const alchemyUrl = `https://eth-mainnet.alchemyapi.io/v2/${process.env.ALCHEMY_API_KEY}/getNFTMetadata`;
      const response = await axios.get(alchemyUrl, {
        params: {
          contractAddress: contract_address,
          tokenId: token_id,
          tokenType: 'ERC721'
        }
      });
      
      const data = response.data;
      
      if (!data || !data.metadata || !data.metadata.attributes) {
        return "This NFT doesn't have attribute metadata or the collection isn't supported";
      }
      
      // Fetch collection stats from Alchemy for rarity comparison
      // This is a simplified approach - a full implementation would require
      // fetching and analyzing the entire collection
      const statsUrl = `https://eth-mainnet.alchemyapi.io/v2/${process.env.ALCHEMY_API_KEY}/getContractMetadata`;
      const statsResponse = await axios.get(statsUrl, {
        params: {
          contractAddress: contract_address
        }
      });
      
      const collectionName = statsResponse.data?.contractMetadata?.name || "Unknown Collection";
      const totalSupply = statsResponse.data?.contractMetadata?.totalSupply || "Unknown";
      
      // For this example, we'll make an approximation of rarity
      // A full implementation would need to analyze all tokens in the collection
      const attributes = data.metadata.attributes;
      
      // For demonstration, we'll use predefined rarity scores for common traits
      // In a real application, you would calculate these from the entire collection
      const rarityMap = {
        'Background': {
          'Blue': 0.30,
          'Red': 0.25,
          'Green': 0.20,
          'Yellow': 0.15,
          'Purple': 0.07,
          'Black': 0.03
        },
        'Eyes': {
          'Regular': 0.50,
          'Sleepy': 0.25,
          'Angry': 0.15,
          'Laser': 0.10
        },
        'Mouth': {
          'Smile': 0.40,
          'Neutral': 0.30,
          'Frown': 0.20,
          'Gold Teeth': 0.10
        }
      };
      
      // Calculate rarity score for this NFT
      let totalRarityScore = 0;
      let traitAnalysis = [];
      
      attributes.forEach(attr => {
        if (!attr.trait_type || attr.value === undefined) return;
        
        const traitType = attr.trait_type;
        const traitValue = String(attr.value);
        
        let rarityScore;
        
        if (rarityMap[traitType] && rarityMap[traitType][traitValue]) {
          rarityScore = 1 / rarityMap[traitType][traitValue];
        } else {
          // If we don't have data for this trait, assume it's of average rarity
          rarityScore = 3;
        }
        
        traitAnalysis.push({
          trait_type: traitType,
          value: traitValue,
          rarity_score: rarityScore.toFixed(2),
          estimated_frequency: rarityMap[traitType]?.[traitValue] ? 
            `${(rarityMap[traitType][traitValue] * 100).toFixed(1)}%` : 
            "Unknown"
        });
        
        totalRarityScore += rarityScore;
      });
      
      // Format the response
      let result = `Trait Rarity Analysis for ${collectionName} - Token ID: ${token_id}\n\n`;
      
      result += `Total Traits: ${attributes.length}\n`;
      result += `Overall Rarity Score: ${totalRarityScore.toFixed(2)}\n\n`;
      
      result += "Individual Trait Analysis:\n";
      traitAnalysis.forEach(trait => {
        result += `- ${trait.trait_type}: ${trait.value}\n`;
        result += `  Rarity Score: ${trait.rarity_score}\n`;
        result += `  Estimated Frequency: ${trait.estimated_frequency}\n`;
      });
      
      result += "\nNote: This is a simplified rarity analysis based on estimated trait frequencies. For accurate rarity ranking, consider using a specialized NFT rarity tool that analyzes the entire collection.";
      
      return result;
    } catch (error) {
      return `Error analyzing trait rarity: ${error.message}`;
    }
  }
};

// Add this tool to your agent's tools list
```

## Step 5: Add Floor Price Tracking

Let's add a tool to track floor prices and sales activity:

```javascript
const trackFloorPriceTool = {
  name: 'track_floor_price',
  description: 'Track floor price and sales activity for a collection',
  parameters: {
    type: 'object',
    properties: {
      contract_address: {
        type: 'string',
        description: 'NFT contract address'
      }
    },
    required: ['contract_address']
  },
  handler: async (params) => {
    try {
      const { contract_address } = params;
      
      // Validate address
      if (!ethers.utils.isAddress(contract_address)) {
        return "Invalid Ethereum contract address provided";
      }
      
      // In a full implementation, you would fetch historical floor prices
      // from a service like OpenSea, NFTScan, or a custom indexer.
      // For this tutorial, we'll simulate some data.
      
      // Create contract instance to get basic info
      const nftContract = new ethers.Contract(contract_address, ERC721_ABI, provider);
      const name = await nftContract.name();
      
      // Simulated floor price data (in ETH)
      const floorPriceHistory = [
        { date: '2023-01-01', price: 1.2 },
        { date: '2023-02-01', price: 1.5 },
        { date: '2023-03-01', price: 1.3 },
        { date: '2023-04-01', price: 1.7 },
        { date: '2023-05-01', price: 2.1 },
        { date: '2023-06-01', price: 1.9 },
        { date: '2023-07-01', price: 2.2 },
        { date: '2023-08-01', price: 2.5 },
        { date: '2023-09-01', price: 2.3 },
        { date: '2023-10-01', price: 2.8 },
        { date: '2023-11-01', price: 3.0 },
        { date: '2023-12-01', price: 2.7 }
      ];
      
      // Simulated recent sales
      const recentSales = [
        { date: '2023-12-15', tokenId: 123, price: 2.8 },
        { date: '2023-12-10', tokenId: 456, price: 3.2 },
        { date: '2023-12-05', tokenId: 789, price: 2.5 },
        { date: '2023-12-01', tokenId: 234, price: 2.7 },
        { date: '2023-11-28', tokenId: 567, price: 3.1 }
      ];
      
      // Calculate floor price statistics
      const currentFloor = floorPriceHistory[floorPriceHistory.length - 1].price;
      const previousFloor = floorPriceHistory[floorPriceHistory.length - 2].price;
      const floorPriceChange = ((currentFloor - previousFloor) / previousFloor) * 100;
      
      const avgSalePrice = recentSales.reduce((sum, sale) => sum + sale.price, 0) / recentSales.length;
      
      // Generate a simple floor price chart
      // In a real application, you would use a library like Chart.js to create this visualization
      const canvas = createCanvas(800, 400);
      const ctx = canvas.getContext('2d');
      
      // Set background
      ctx.fillStyle = '#f8f9fa';
      ctx.fillRect(0, 0, 800, 400);
      
      // Draw chart axes
      ctx.strokeStyle = '#212529';
      ctx.lineWidth = 2;
      ctx.beginPath();
      ctx.moveTo(50, 350);
      ctx.lineTo(750, 350);
      ctx.stroke();
      
      ctx.beginPath();
      ctx.moveTo(50, 50);
      ctx.lineTo(50, 350);
      ctx.stroke();
      
      // Draw title
      ctx.font = 'bold 20px Arial';
      ctx.fillStyle = '#212529';
      ctx.fillText(`${name} - Floor Price History`, 300, 30);
      
      // Draw floor price data
      const maxPrice = Math.max(...floorPriceHistory.map(d => d.price)) * 1.2;
      const xStep = 700 / (floorPriceHistory.length - 1);
      
      ctx.strokeStyle = '#0d6efd';
      ctx.lineWidth = 3;
      ctx.beginPath();
      
      floorPriceHistory.forEach((dataPoint, i) => {
        const x = 50 + i * xStep;
        const y = 350 - (dataPoint.price / maxPrice) * 300;
        
        if (i === 0) {
          ctx.moveTo(x, y);
        } else {
          ctx.lineTo(x, y);
        }
        
        // Draw data point
        ctx.fillStyle = '#0d6efd';
        ctx.beginPath();
        ctx.arc(x, y, 5, 0, Math.PI * 2);
        ctx.fill();
        
        // Draw x-axis label (simplified)
        if (i % 3 === 0) {
          ctx.fillStyle = '#212529';
          ctx.font = '12px Arial';
          ctx.fillText(dataPoint.date.split('-')[1], x - 10, 370);
        }
      });
      
      ctx.stroke();
      
      // Draw y-axis labels
      for (let i = 0; i <= 5; i++) {
        const y = 350 - (i / 5) * 300;
        const price = (i / 5) * maxPrice;
        
        ctx.fillStyle = '#212529';
        ctx.font = '12px Arial';
        ctx.fillText(price.toFixed(1) + ' ETH', 10, y + 5);
        
        // Draw horizontal grid line
        ctx.strokeStyle = '#dee2e6';
        ctx.lineWidth = 1;
        ctx.beginPath();
        ctx.moveTo(50, y);
        ctx.lineTo(750, y);
        ctx.stroke();
      }
      
      // Save chart to file
      const chartFilename = `floor_price_${contract_address.substring(0, 10)}.png`;
      const out = fs.createWriteStream(chartFilename);
      const stream = canvas.createPNGStream();
      stream.pipe(out);
      
      // Format the response
      let result = `Floor Price Analysis for ${name}\n\n`;
      
      result += `Current Floor Price: ${currentFloor} ETH\n`;
      result += `Monthly Change: ${floorPriceChange.toFixed(2)}%\n`;
      result += `Average Recent Sale Price: ${avgSalePrice.toFixed(2)} ETH\n\n`;
      
      result += "Recent Sales:\n";
      recentSales.forEach(sale => {
        result += `- ${sale.date}: Token #${sale.tokenId} sold for ${sale.price} ETH\n`;
      });
      
      result += `\nFloor price chart saved as ${chartFilename}`;
      
      return result;
    } catch (error) {
      return `Error tracking floor price: ${error.message}`;
    }
  }
};

// Add this tool to your agent's tools list
```

## Step 6: Add NFT Price Estimation Tool

Let's add a tool to estimate the price of an NFT based on its traits:

```javascript
const estimateNFTPriceTool = {
  name: 'estimate_nft_price',
  description: 'Estimate the price of an NFT based on its traits and collection data',
  parameters: {
    type: 'object',
    properties: {
      contract_address: {
        type: 'string',
        description: 'NFT contract address'
      },
      token_id: {
        type: 'integer',
        description: 'Token ID of the NFT'
      }
    },
    required: ['contract_address', 'token_id']
  },
  handler: async (params) => {
    try {
      const { contract_address, token_id } = params;
      
      // Fetch NFT metadata (similar to analyzeTraitRarityTool)
      // We'll reuse the token fetching code and add price estimation logic
      if (!process.env.ALCHEMY_API_KEY) {
        return "This feature requires an Alchemy API key";
      }
      
      const alchemyUrl = `https://eth-mainnet.alchemyapi.io/v2/${process.env.ALCHEMY_API_KEY}/getNFTMetadata`;
      const response = await axios.get(alchemyUrl, {
        params: {
          contractAddress: contract_address,
          tokenId: token_id,
          tokenType: 'ERC721'
        }
      });
      
      const data = response.data;
      
      if (!data || !data.metadata || !data.metadata.attributes) {
        return "This NFT doesn't have attribute metadata or the collection isn't supported";
      }
      
      const attributes = data.metadata.attributes;
      const title = data.title || `Unknown NFT #${token_id}`;
      
      // Get collection name and basic info
      const nftContract = new ethers.Contract(contract_address, ERC721_ABI, provider);
      const collectionName = await nftContract.name();
      
      // In a full implementation, you would:
      // 1. Fetch historical sales for this collection
      // 2. Analyze trait value impact on prices
      // 3. Use machine learning to predict prices
      
      // For this tutorial, we'll use a simplified model with simulated data
      
      // Simulated collection floor price
      const floorPrice = 2.5; // ETH
      
      // Simulated trait multipliers (in a real app, these would be calculated from sales data)
      const traitMultipliers = {
        'Background': {
          'Blue': 1.0,
          'Red': 1.2,
          'Green': 0.9,
          'Yellow': 1.1,
          'Purple': 1.5,
          'Black': 2.0
        },
        'Eyes': {
          'Regular': 1.0,
          'Sleepy': 1.2,
          'Angry': 1.4,
          'Laser': 2.2
        },
        'Mouth': {
          'Smile': 1.0,
          'Neutral': 0.9,
          'Frown': 1.1,
          'Gold Teeth': 1.8
        }
      };
      
      // Calculate base price multiplier from traits
      let priceMultiplier = 1.0;
      
      attributes.forEach(attr => {
        if (!attr.trait_type || attr.value === undefined) return;
        
        const traitType = attr.trait_type;
        const traitValue = String(attr.value);
        
        if (traitMultipliers[traitType] && traitMultipliers[traitType][traitValue]) {
          priceMultiplier *= traitMultipliers[traitType][traitValue];
        }
      });
      
      // Calculate estimated price
      const estimatedPrice = floorPrice * priceMultiplier;
      
      // Calculate confidence interval (simplified)
      const lowerBound = estimatedPrice * 0.8;
      const upperBound = estimatedPrice * 1.2;
      
      // Format the response
      let result = `NFT Price Estimate for ${title} from ${collectionName}\n\n`;
      
      result += `Estimated Price: ${estimatedPrice.toFixed(2)} ETH\n`;
      result += `Price Range: ${lowerBound.toFixed(2)} - ${upperBound.toFixed(2)} ETH\n`;
      result += `Collection Floor Price: ${floorPrice} ETH\n`;
      result += `Price Multiplier: ${priceMultiplier.toFixed(2)}x\n\n`;
      
      result += "Trait Value Analysis:\n";
      attributes.forEach(attr => {
        if (!attr.trait_type || attr.value === undefined) return;
        
        const traitType = attr.trait_type;
        const traitValue = String(attr.value);
        
        const multiplier = traitMultipliers[traitType]?.[traitValue] || 1.0;
        const impact = ((multiplier - 1) * 100).toFixed(1);
        
        result += `- ${traitType}: ${traitValue}\n`;
        result += `  Price Impact: ${impact}%\n`;
      });
      
      result += "\nNote: This price estimate is based on simulated data and should be used for educational purposes only. Many factors affect NFT prices including market conditions, artist reputation, and collector demand.";
      
      return result;
    } catch (error) {
      return `Error estimating NFT price: ${error.message}`;
    }
  }
};

// Add this tool to your agent's tools list
```

## Step 7: Putting It All Together

Update your agent initialization to include all the tools:

```javascript
// Initialize the Alith agent with all tools
const agent = new Agent({
  name: 'NFT Analyzer',
  model: 'gpt-4',
  preamble: `You are an NFT analysis assistant powered by AI. You help users understand 
  NFT collections, analyze traits, evaluate rarity, and provide insights about digital assets. 
  You can look up NFT metadata, check ownership, and analyze collections.
  
  When providing information:
  - Explain NFT concepts clearly for both beginners and experts
  - Provide objective analysis of collections and traits
  - Describe how rarity and traits may impact value
  - Clarify that your analysis is not financial advice`,
  tools: [
    getNFTOwnershipTool,
    getNFTMetadataTool,
    getCollectionInfoTool,
    analyzeTraitRarityTool,
    trackFloorPriceTool,
    estimateNFTPriceTool
  ]
});
```

## Step 8: Running Your NFT Analyzer

Run your NFT analyzer:

```bash
node analyzer.js
```

You can now interact with your analyzer using natural language:

- "What NFTs does address 0x1234...5678 own from the BAYC collection?"
- "Show me the metadata for BAYC #1234"
- "Analyze the rarity of CryptoPunk #5822"
- "What's the floor price history for Azuki?"
- "Estimate the price of Doodles #8888"
- "Tell me about the Moonbirds collection"

## Step 9: Building a Web Interface (Optional)

For a better user experience, you could create a web interface:

```bash
npm install express cors
```

Create a file named `server.js`:

```javascript
const express = require('express');
const cors = require('cors');
const { agent } = require('./analyzer');

const app = express();
const port = 3000;

app.use(cors());
app.use(express.json());
app.use(express.static('public'));
app.use('/charts', express.static('./'));

app.post('/api/analyze', async (req, res) => {
  try {
    const { query } = req.body;
    const response = await agent.prompt(query);
    res.json({ response });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

app.listen(port, () => {
  console.log(`NFT Analyzer server running at http://localhost:${port}`);
});
```

## Step 10: Extending Your Analyzer

Here are some ways to extend your NFT analyzer:

1. **Multi-chain Support**: Add support for NFTs on Polygon, Solana, Flow, and other blockchains
2. **Collection Comparison**: Create tools to compare different NFT collections
3. **Wash Trading Detection**: Implement algorithms to detect suspicious trading patterns
4. **Sentiment Analysis**: Analyze social media sentiment around collections
5. **NFT Portfolio Tracker**: Track users' NFT portfolios and their value over time
6. **Rarity Sniping**: Create tools to find undervalued NFTs based on trait rarity
7. **Historical Analysis**: Track how rarity perception and valuation change over time

## Conclusion

You've built a powerful NFT analyzer with Alith that can evaluate collections, analyze trait rarity, track floor prices, and estimate NFT values. This application demonstrates how AI agents can help users navigate the complex world of digital collectibles.

By combining on-chain data, collection analytics, and natural language processing, you've created an assistant that can help collectors make more informed decisions about NFTs. 