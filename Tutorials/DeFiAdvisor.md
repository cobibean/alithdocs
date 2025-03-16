# Building a DeFi Portfolio Advisor with Alith

This tutorial guides you through creating an AI-powered DeFi portfolio advisor that can analyze a user's cryptocurrency holdings, suggest optimizations, and provide personalized investment advice.

## What You'll Learn

- Connecting to DeFi protocols and fetching on-chain data
- Creating specialized financial analysis tools
- Building an agent that can provide investment advice
- Personalizing recommendations based on risk profiles
- Calculating portfolio metrics and optimization strategies

## Prerequisites

- Node.js v16+ installed
- Basic JavaScript knowledge 
- OpenAI or Anthropic API key
- Ethereum node access (via Infura, Alchemy, etc.)

## Step 1: Set Up Your Project

Create a new directory and initialize your project:

```bash
mkdir defi-advisor
cd defi-advisor
npm init -y
```

Install the required dependencies:

```bash
npm install alith dotenv ethers axios web3 @uniswap/sdk-core @uniswap/v3-sdk
```

Create a `.env` file for your API keys:

```
OPENAI_API_KEY=your_openai_key_here
INFURA_API_KEY=your_infura_key_here
COINGECKO_API_KEY=your_coingecko_key_here
```

## Step 2: Create Basic Portfolio Analyzer

Create a file named `advisor.js` with the following basic structure:

```javascript
require('dotenv').config();
const { Agent } = require('alith');
const { ethers } = require('ethers');
const axios = require('axios');

// Initialize Ethereum provider
const provider = new ethers.providers.JsonRpcProvider(
  `https://mainnet.infura.io/v3/${process.env.INFURA_API_KEY}`
);

// ERC20 ABI - just the functions we need
const ERC20_ABI = [
  "function balanceOf(address owner) view returns (uint256)",
  "function decimals() view returns (uint8)",
  "function symbol() view returns (string)",
  "function name() view returns (string)"
];

// Create a function to get token balances
async function getTokenBalances(address, tokenAddresses) {
  const balances = [];
  
  for (const tokenAddress of tokenAddresses) {
    try {
      const tokenContract = new ethers.Contract(tokenAddress, ERC20_ABI, provider);
      const decimals = await tokenContract.decimals();
      const symbol = await tokenContract.symbol();
      const name = await tokenContract.name();
      const balance = await tokenContract.balanceOf(address);
      
      // Convert to human readable format
      const balanceFormatted = ethers.utils.formatUnits(balance, decimals);
      
      if (parseFloat(balanceFormatted) > 0) {
        balances.push({
          token: name,
          symbol,
          address: tokenAddress,
          balance: balanceFormatted
        });
      }
    } catch (error) {
      console.error(`Error fetching balance for token ${tokenAddress}:`, error.message);
    }
  }
  
  return balances;
}

// Create a tool to check ETH balance
const getEthBalanceTool = {
  name: 'get_eth_balance',
  description: 'Get the ETH balance of an Ethereum address',
  parameters: {
    type: 'object',
    properties: {
      address: {
        type: 'string',
        description: 'Ethereum address to check'
      }
    },
    required: ['address']
  },
  handler: async (params) => {
    try {
      const balance = await provider.getBalance(params.address);
      const ethBalance = ethers.utils.formatEther(balance);
      return `ETH Balance: ${ethBalance} ETH`;
    } catch (error) {
      return `Error: ${error.message}`;
    }
  }
};

// Create a tool to get token prices
const getTokenPricesTool = {
  name: 'get_token_prices',
  description: 'Get current prices for specified tokens',
  parameters: {
    type: 'object',
    properties: {
      tokens: {
        type: 'array',
        items: {
          type: 'string'
        },
        description: 'List of token symbols (e.g., ["BTC", "ETH", "LINK"])'
      }
    },
    required: ['tokens']
  },
  handler: async (params) => {
    try {
      const tokens = params.tokens.join(',').toLowerCase();
      const response = await axios.get(
        `https://api.coingecko.com/api/v3/simple/price?ids=${tokens}&vs_currencies=usd`
      );
      
      let result = "Current Token Prices:\n";
      
      for (const [token, data] of Object.entries(response.data)) {
        if (data.usd) {
          result += `${token.toUpperCase()}: $${data.usd}\n`;
        }
      }
      
      return result;
    } catch (error) {
      return `Error fetching prices: ${error.message}`;
    }
  }
};

// Create a tool to analyze a portfolio
const analyzePortfolioTool = {
  name: 'analyze_portfolio',
  description: 'Analyze a crypto portfolio including ETH and common ERC20 tokens',
  parameters: {
    type: 'object',
    properties: {
      address: {
        type: 'string',
        description: 'Ethereum address to analyze'
      }
    },
    required: ['address']
  },
  handler: async (params) => {
    try {
      const address = params.address;
      
      // Get ETH balance
      const ethBalance = await provider.getBalance(address);
      const ethBalanceFormatted = ethers.utils.formatEther(ethBalance);
      
      // Common ERC20 tokens to check
      const commonTokens = [
        '0xdAC17F958D2ee523a2206206994597C13D831ec7', // USDT
        '0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48', // USDC
        '0x6B175474E89094C44Da98b954EedeAC495271d0F', // DAI
        '0x2260FAC5E5542a773Aa44fBCfeDf7C193bc2C599', // WBTC
        '0x1f9840a85d5aF5bf1D1762F925BDADdC4201F984', // UNI
        '0x514910771AF9Ca656af840dff83E8264EcF986CA', // LINK
        '0x7D1AfA7B718fb893dB30A3aBc0Cfc608AaCfeBB0', // MATIC
        '0x95aD61b0a150d79219dCF64E1E6Cc01f0B64C4cE'  // SHIB
      ];
      
      // Get token balances
      const tokenBalances = await getTokenBalances(address, commonTokens);
      
      // Get prices for ETH and found tokens
      const tokenSymbols = ['ethereum', ...tokenBalances.map(t => t.symbol.toLowerCase())];
      const priceResponse = await axios.get(
        `https://api.coingecko.com/api/v3/simple/price?ids=${tokenSymbols.join(',')}&vs_currencies=usd`
      );
      
      // Calculate portfolio value
      let totalValue = 0;
      let portfolioBreakdown = [];
      
      // Add ETH to portfolio
      const ethPrice = priceResponse.data.ethereum?.usd || 0;
      const ethValue = parseFloat(ethBalanceFormatted) * ethPrice;
      totalValue += ethValue;
      
      if (parseFloat(ethBalanceFormatted) > 0) {
        portfolioBreakdown.push({
          asset: 'Ethereum',
          symbol: 'ETH',
          balance: ethBalanceFormatted,
          price: ethPrice,
          value: ethValue
        });
      }
      
      // Add tokens to portfolio
      for (const token of tokenBalances) {
        const tokenPrice = priceResponse.data[token.symbol.toLowerCase()]?.usd || 0;
        const tokenValue = parseFloat(token.balance) * tokenPrice;
        totalValue += tokenValue;
        
        portfolioBreakdown.push({
          asset: token.token,
          symbol: token.symbol,
          balance: token.balance,
          price: tokenPrice,
          value: tokenValue
        });
      }
      
      // Sort by value (descending)
      portfolioBreakdown.sort((a, b) => b.value - a.value);
      
      // Calculate percentages
      portfolioBreakdown = portfolioBreakdown.map(asset => ({
        ...asset,
        percentage: (asset.value / totalValue * 100).toFixed(2) + '%'
      }));
      
      // Format results
      let result = `Portfolio Analysis for ${address}\n\n`;
      result += `Total Portfolio Value: $${totalValue.toFixed(2)}\n\n`;
      result += `Asset Breakdown:\n`;
      
      portfolioBreakdown.forEach(asset => {
        result += `- ${asset.asset} (${asset.symbol}): ${asset.balance} tokens × $${asset.price} = $${asset.value.toFixed(2)} (${asset.percentage})\n`;
      });
      
      return result;
    } catch (error) {
      return `Error analyzing portfolio: ${error.message}`;
    }
  }
};

// Initialize the Alith agent
const agent = new Agent({
  name: 'DeFi Advisor',
  model: 'gpt-4',
  preamble: `You are a DeFi portfolio advisor powered by AI. You help users understand and optimize their 
  cryptocurrency portfolios with data-driven insights and personalized recommendations. You provide advice 
  on portfolio balancing, risk management, and potential opportunities.

  When giving advice:
  - Always consider risk tolerance and diversification
  - Explain the reasoning behind your suggestions
  - Provide both conservative and growth-oriented options
  - Highlight potential risks of different strategies
  - Remind users that all crypto investments carry risk
  - Clarify that your advice is not financial advice`,
  tools: [
    getEthBalanceTool,
    getTokenPricesTool,
    analyzePortfolioTool
  ]
});

// Create a simple command-line interface
async function main() {
  console.log("DeFi Portfolio Advisor powered by Alith");
  console.log("Type 'exit' to quit");
  
  const readline = require('readline').createInterface({
    input: process.stdin,
    output: process.stdout
  });
  
  const askQuestion = () => {
    readline.question("\nHow can I help with your crypto portfolio? ", async (query) => {
      if (query.toLowerCase() === 'exit') {
        console.log("Thank you for using the DeFi Portfolio Advisor. Goodbye!");
        readline.close();
        return;
      }
      
      try {
        console.log("Analyzing...");
        const response = await agent.prompt(query);
        console.log("\nAdvisor Response:");
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

## Step 3: Add Risk Profile Analysis

Let's add a tool to analyze a user's risk profile:

```javascript
const assessRiskProfileTool = {
  name: 'assess_risk_profile',
  description: 'Assess an investment risk profile based on portfolio composition',
  parameters: {
    type: 'object',
    properties: {
      address: {
        type: 'string',
        description: 'Ethereum address to analyze'
      }
    },
    required: ['address']
  },
  handler: async (params) => {
    try {
      const address = params.address;
      
      // Get portfolio data (reusing code from analyzePortfolioTool)
      const ethBalance = await provider.getBalance(address);
      const ethBalanceFormatted = ethers.utils.formatEther(ethBalance);
      
      const commonTokens = [
        '0xdAC17F958D2ee523a2206206994597C13D831ec7', // USDT
        '0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48', // USDC
        '0x6B175474E89094C44Da98b954EedeAC495271d0F', // DAI
        '0x2260FAC5E5542a773Aa44fBCfeDf7C193bc2C599', // WBTC
        '0x1f9840a85d5aF5bf1D1762F925BDADdC4201F984', // UNI
        '0x514910771AF9Ca656af840dff83E8264EcF986CA', // LINK
        '0x7D1AfA7B718fb893dB30A3aBc0Cfc608AaCfeBB0', // MATIC
        '0x95aD61b0a150d79219dCF64E1E6Cc01f0B64C4cE'  // SHIB
      ];
      
      const tokenBalances = await getTokenBalances(address, commonTokens);
      
      const tokenSymbols = ['ethereum', ...tokenBalances.map(t => t.symbol.toLowerCase())];
      const priceResponse = await axios.get(
        `https://api.coingecko.com/api/v3/simple/price?ids=${tokenSymbols.join(',')}&vs_currencies=usd`
      );
      
      // Calculate portfolio value and composition
      let totalValue = 0;
      let stablecoinsValue = 0;
      let blueChipsValue = 0;
      let altcoinsValue = 0;
      
      // Process ETH (considered blue chip)
      const ethPrice = priceResponse.data.ethereum?.usd || 0;
      const ethValue = parseFloat(ethBalanceFormatted) * ethPrice;
      totalValue += ethValue;
      blueChipsValue += ethValue;
      
      // Process tokens
      for (const token of tokenBalances) {
        const tokenPrice = priceResponse.data[token.symbol.toLowerCase()]?.usd || 0;
        const tokenValue = parseFloat(token.balance) * tokenPrice;
        totalValue += tokenValue;
        
        // Categorize tokens
        if (['USDT', 'USDC', 'DAI', 'BUSD', 'TUSD'].includes(token.symbol)) {
          stablecoinsValue += tokenValue;
        } else if (['WBTC', 'ETH', 'BTC', 'WETH'].includes(token.symbol)) {
          blueChipsValue += tokenValue;
        } else {
          altcoinsValue += tokenValue;
        }
      }
      
      // Calculate percentages
      const stablecoinsPercentage = totalValue > 0 ? (stablecoinsValue / totalValue) * 100 : 0;
      const blueChipsPercentage = totalValue > 0 ? (blueChipsValue / totalValue) * 100 : 0;
      const altcoinsPercentage = totalValue > 0 ? (altcoinsValue / totalValue) * 100 : 0;
      
      // Determine risk profile
      let riskProfile;
      let explanation;
      
      if (stablecoinsPercentage >= 70) {
        riskProfile = "Very Conservative";
        explanation = "Portfolio is heavily weighted toward stablecoins (70%+), indicating a very risk-averse approach.";
      } else if (stablecoinsPercentage >= 50) {
        riskProfile = "Conservative";
        explanation = "Portfolio has a significant stablecoin allocation (50%+), suggesting a conservative strategy.";
      } else if (blueChipsPercentage >= 60) {
        riskProfile = "Moderate";
        explanation = "Portfolio focuses on established cryptocurrencies like BTC and ETH, indicating a balanced approach.";
      } else if (blueChipsPercentage >= 40) {
        riskProfile = "Growth-Oriented";
        explanation = "Portfolio has a healthy mix of established coins and other assets, suggesting comfort with moderate risk.";
      } else if (altcoinsPercentage >= 50) {
        riskProfile = "Aggressive";
        explanation = "Portfolio has significant allocation to altcoins, indicating comfort with higher volatility and risk.";
      } else if (altcoinsPercentage >= 70) {
        riskProfile = "Very Aggressive";
        explanation = "Portfolio is heavily weighted toward altcoins (70%+), showing a high-risk, high-reward strategy.";
      } else {
        riskProfile = "Balanced";
        explanation = "Portfolio has a mix of stablecoins, blue chips, and altcoins without strong concentration in any category.";
      }
      
      return `
Risk Profile Assessment for ${address}:

Profile: ${riskProfile}
${explanation}

Portfolio Composition:
- Stablecoins: ${stablecoinsPercentage.toFixed(2)}%
- Blue Chips (BTC, ETH): ${blueChipsPercentage.toFixed(2)}%
- Altcoins: ${altcoinsPercentage.toFixed(2)}%

Total Portfolio Value: $${totalValue.toFixed(2)}
      `.trim();
    } catch (error) {
      return `Error assessing risk profile: ${error.message}`;
    }
  }
};

// Add this tool to your agent's tools list
```

## Step 4: Add DeFi Protocol Integration

Let's add a tool to check Uniswap liquidity positions:

```javascript
const { Pool } = require('@uniswap/v3-sdk');
const { Token } = require('@uniswap/sdk-core');

// Add this for Uniswap v3 pool ABI
const UNISWAP_V3_POOL_ABI = [
  "function token0() external view returns (address)",
  "function token1() external view returns (address)",
  "function fee() external view returns (uint24)",
  "function positions(bytes32) external view returns (uint128, uint256, uint256, uint128, uint128)"
];

// Add this function for looking up positions
async function getUniswapPositions(address) {
  try {
    // Common Uniswap v3 pools to check (this is just a small sample)
    const commonPools = [
      '0x8ad599c3A0ff1De082011EFDDc58f1908eb6e6D8', // USDC/ETH 0.3%
      '0x88e6A0c2dDD26FEEb64F039a2c41296FcB3f5640', // USDC/ETH 0.05%
      '0x7BeA39867e4169DBe237d55C8242a8f2fcDcc387', // ETH/USDT 0.3%
      '0x4e68Ccd3E89f51C3074ca5072bbAC773960dFa36', // ETH/WBTC 0.3%
    ];
    
    const positions = [];
    
    for (const poolAddress of commonPools) {
      const poolContract = new ethers.Contract(poolAddress, UNISWAP_V3_POOL_ABI, provider);
      
      // Get pool info
      const token0Address = await poolContract.token0();
      const token1Address = await poolContract.token1();
      const fee = await poolContract.fee();
      
      // Get token info
      const token0Contract = new ethers.Contract(token0Address, ERC20_ABI, provider);
      const token1Contract = new ethers.Contract(token1Address, ERC20_ABI, provider);
      
      const token0Symbol = await token0Contract.symbol();
      const token1Symbol = await token1Contract.symbol();
      
      // Check if user has liquidity position in this pool
      // This is a simplified approach - in a real app, you'd need to check NFT positions
      const positionKey = ethers.utils.keccak256(
        ethers.utils.solidityPack(
          ['address', 'int24', 'int24'],
          [address, -887220, 887220] // Using the full range as a placeholder
        )
      );
      
      try {
        const position = await poolContract.positions(positionKey);
        
        // If liquidity > 0, user has a position
        if (position.liquidity.gt(0)) {
          positions.push({
            pool: `${token0Symbol}/${token1Symbol} ${fee/10000}%`,
            address: poolAddress,
            liquidity: position.liquidity.toString()
          });
        }
      } catch (error) {
        console.error(`Error checking position in pool ${poolAddress}:`, error.message);
      }
    }
    
    return positions;
  } catch (error) {
    console.error("Error getting Uniswap positions:", error.message);
    return [];
  }
}

const checkDeFiPositionsTool = {
  name: 'check_defi_positions',
  description: 'Check DeFi positions including Uniswap liquidity',
  parameters: {
    type: 'object',
    properties: {
      address: {
        type: 'string',
        description: 'Ethereum address to check'
      }
    },
    required: ['address']
  },
  handler: async (params) => {
    try {
      const address = params.address;
      
      // Get Uniswap positions
      const uniswapPositions = await getUniswapPositions(address);
      
      let result = `DeFi Positions for ${address}:\n\n`;
      
      // Uniswap positions
      result += "Uniswap Liquidity Positions:\n";
      if (uniswapPositions.length > 0) {
        uniswapPositions.forEach(position => {
          result += `- ${position.pool} (Pool: ${position.address})\n`;
          result += `  Liquidity: ${position.liquidity}\n`;
        });
      } else {
        result += "No Uniswap V3 liquidity positions found.\n";
      }
      
      // Note: In a full implementation, you would check other protocols
      // like Compound, Aave, Curve, etc.
      
      result += "\nNote: This is a simplified check. A full scan would require checking all DeFi protocols.";
      
      return result;
    } catch (error) {
      return `Error checking DeFi positions: ${error.message}`;
    }
  }
};

// Add this tool to your agent's tools list
```

## Step 5: Add Portfolio Optimization Recommendations

Let's add a tool to generate portfolio optimization suggestions:

```javascript
const generateOptimizationTool = {
  name: 'generate_portfolio_optimization',
  description: 'Generate portfolio optimization recommendations based on risk profile and market conditions',
  parameters: {
    type: 'object',
    properties: {
      address: {
        type: 'string',
        description: 'Ethereum address to analyze'
      },
      risk_profile: {
        type: 'string',
        description: 'Risk profile (e.g., "Conservative", "Moderate", "Aggressive")'
      }
    },
    required: ['address', 'risk_profile']
  },
  handler: async (params) => {
    try {
      const address = params.address;
      const riskProfile = params.risk_profile;
      
      // Get current portfolio
      const ethBalance = await provider.getBalance(address);
      const ethBalanceFormatted = ethers.utils.formatEther(ethBalance);
      
      const commonTokens = [
        '0xdAC17F958D2ee523a2206206994597C13D831ec7', // USDT
        '0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48', // USDC
        '0x6B175474E89094C44Da98b954EedeAC495271d0F', // DAI
        '0x2260FAC5E5542a773Aa44fBCfeDf7C193bc2C599', // WBTC
        '0x1f9840a85d5aF5bf1D1762F925BDADdC4201F984', // UNI
        '0x514910771AF9Ca656af840dff83E8264EcF986CA', // LINK
        '0x7D1AfA7B718fb893dB30A3aBc0Cfc608AaCfeBB0', // MATIC
      ];
      
      const tokenBalances = await getTokenBalances(address, commonTokens);
      
      const tokenSymbols = ['ethereum', ...tokenBalances.map(t => t.symbol.toLowerCase())];
      const priceResponse = await axios.get(
        `https://api.coingecko.com/api/v3/simple/price?ids=${tokenSymbols.join(',')}&vs_currencies=usd`
      );
      
      // Calculate current allocations
      let totalValue = 0;
      let currentAllocation = {};
      
      // Add ETH
      const ethPrice = priceResponse.data.ethereum?.usd || 0;
      const ethValue = parseFloat(ethBalanceFormatted) * ethPrice;
      totalValue += ethValue;
      currentAllocation['ETH'] = ethValue;
      
      // Add tokens
      for (const token of tokenBalances) {
        const tokenPrice = priceResponse.data[token.symbol.toLowerCase()]?.usd || 0;
        const tokenValue = parseFloat(token.balance) * tokenPrice;
        totalValue += tokenValue;
        currentAllocation[token.symbol] = tokenValue;
      }
      
      // Convert to percentages
      Object.keys(currentAllocation).forEach(symbol => {
        currentAllocation[symbol] = (currentAllocation[symbol] / totalValue * 100).toFixed(2);
      });
      
      // Define target allocations based on risk profile
      let targetAllocation = {};
      let optimizationSteps = [];
      
      switch (riskProfile.toLowerCase()) {
        case 'very conservative':
          targetAllocation = {
            'Stablecoins': '70-80%',
            'ETH/BTC': '15-25%',
            'Other': '0-10%'
          };
          optimizationSteps = [
            "Increase stablecoin allocation to at least 70% of portfolio",
            "Hold ETH/BTC as the primary volatile assets",
            "Minimize exposure to smaller altcoins",
            "Consider stablecoin yield farming for safe returns"
          ];
          break;
          
        case 'conservative':
          targetAllocation = {
            'Stablecoins': '50-60%',
            'ETH/BTC': '30-40%',
            'Blue Chip DeFi': '5-15%',
            'Other': '0-5%'
          };
          optimizationSteps = [
            "Maintain at least 50% in stablecoins",
            "Allocate 30-40% to ETH and BTC",
            "Consider top DeFi tokens like AAVE, UNI for the remainder",
            "Explore conservative yield strategies"
          ];
          break;
          
        case 'moderate':
          targetAllocation = {
            'Stablecoins': '30-40%',
            'ETH/BTC': '40-50%',
            'Blue Chip DeFi': '10-20%',
            'Mid-cap Alts': '5-10%'
          };
          optimizationSteps = [
            "Maintain ETH/BTC as core holdings (40-50%)",
            "Keep 30-40% in stablecoins as safety",
            "Allocate 10-20% to established DeFi protocols",
            "Consider small positions in promising L1/L2 solutions"
          ];
          break;
          
        case 'growth-oriented':
          targetAllocation = {
            'Stablecoins': '20-30%',
            'ETH/BTC': '40-50%',
            'Blue Chip DeFi': '15-25%',
            'Mid-cap Alts': '10-15%'
          };
          optimizationSteps = [
            "Core ETH/BTC position (40-50%)",
            "Reduced stablecoin position (20-30%)",
            "Significant allocation to proven DeFi assets",
            "Strategic positions in growth-potential projects",
            "Consider active yield strategies"
          ];
          break;
          
        case 'aggressive':
          targetAllocation = {
            'Stablecoins': '10-20%',
            'ETH/BTC': '30-40%',
            'Blue Chip DeFi': '20-30%',
            'Mid-cap Alts': '15-25%',
            'Small-cap Opportunities': '5-15%'
          };
          optimizationSteps = [
            "Maintain core ETH/BTC allocation (30-40%)",
            "Limited stablecoin reserves (10-20%)",
            "Significant exposure to high-potential DeFi protocols",
            "Strategic positions in emerging tech and L2 solutions",
            "Consider advanced yield strategies"
          ];
          break;
          
        case 'very aggressive':
          targetAllocation = {
            'Stablecoins': '5-10%',
            'ETH/BTC': '20-30%',
            'Blue Chip DeFi': '20-30%',
            'Mid-cap Alts': '20-30%',
            'Small-cap Opportunities': '10-25%'
          };
          optimizationSteps = [
            "Minimal stablecoin position (5-10%)",
            "Reduced ETH/BTC allocation compared to other profiles",
            "Heavy allocation to high-growth potential assets",
            "Actively managed positions in emerging tech",
            "Consider leveraged positions and advanced strategies"
          ];
          break;
          
        default:
          targetAllocation = {
            'Stablecoins': '30-40%',
            'ETH/BTC': '40-50%',
            'Blue Chip DeFi': '10-20%',
            'Mid-cap Alts': '5-10%'
          };
          optimizationSteps = [
            "Balance your portfolio between ETH/BTC and stablecoins",
            "Add selective exposure to established DeFi protocols",
            "Consider small positions in promising projects"
          ];
      }
      
      // Format results
      let result = `Portfolio Optimization for ${address} (${riskProfile} Profile):\n\n`;
      
      result += `Current Portfolio Allocation:\n`;
      Object.entries(currentAllocation).forEach(([symbol, percentage]) => {
        result += `- ${symbol}: ${percentage}%\n`;
      });
      
      result += `\nRecommended Target Allocation:\n`;
      Object.entries(targetAllocation).forEach(([category, range]) => {
        result += `- ${category}: ${range}\n`;
      });
      
      result += `\nOptimization Steps:\n`;
      optimizationSteps.forEach((step, index) => {
        result += `${index + 1}. ${step}\n`;
      });
      
      result += `\nNote: These recommendations are for informational purposes only and do not constitute financial advice. Always do your own research before making investment decisions.`;
      
      return result;
    } catch (error) {
      return `Error generating optimization: ${error.message}`;
    }
  }
};

// Add this tool to your agent's tools list
```

## Step 6: Putting It All Together

Update your agent initialization to include all the tools:

```javascript
// Initialize the Alith agent with all tools
const agent = new Agent({
  name: 'DeFi Advisor',
  model: 'gpt-4',
  preamble: `You are a DeFi portfolio advisor powered by AI. You help users understand and optimize their 
  cryptocurrency portfolios with data-driven insights and personalized recommendations. You provide advice 
  on portfolio balancing, risk management, and potential opportunities.

  When giving advice:
  - Always consider risk tolerance and diversification
  - Explain the reasoning behind your suggestions
  - Provide both conservative and growth-oriented options
  - Highlight potential risks of different strategies
  - Remind users that all crypto investments carry risk
  - Clarify that your advice is not financial advice`,
  tools: [
    getEthBalanceTool,
    getTokenPricesTool,
    analyzePortfolioTool,
    assessRiskProfileTool,
    checkDeFiPositionsTool,
    generateOptimizationTool
  ]
});
```

## Step 7: Running Your DeFi Advisor

Run your DeFi portfolio advisor:

```bash
node advisor.js
```

You can now interact with your advisor using natural language:

- "Analyze the portfolio for address 0x1234...5678"
- "What's the risk profile of this wallet: 0xabcd...ef00?"
- "How can this portfolio be optimized for moderate risk?"
- "What DeFi positions does address 0x9876...5432 have?"
- "Compare ETH and BTC prices over the last week"

## Step 8: Building a Web Interface (Optional)

For a better user experience, you could create a simple web interface using Express:

```bash
npm install express cors
```

Create a file named `server.js`:

```javascript
const express = require('express');
const cors = require('cors');
const { agent } = require('./advisor');

const app = express();
const port = 3000;

app.use(cors());
app.use(express.json());
app.use(express.static('public'));

app.post('/api/ask', async (req, res) => {
  try {
    const { query } = req.body;
    const response = await agent.prompt(query);
    res.json({ response });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

app.listen(port, () => {
  console.log(`DeFi Advisor server running at http://localhost:${port}`);
});
```

Create a basic HTML interface in a `public` folder.

## Step 9: Extending Your Advisor

Here are some ways to extend your DeFi portfolio advisor:

1. **Historical Analysis**: Add tools to analyze portfolio performance over time
2. **Gas Optimization**: Suggest optimal times for transactions based on gas prices
3. **Tax Reporting**: Add tools to generate tax reports for crypto transactions
4. **Multiple Chains**: Expand to support BSC, Polygon, Solana, and other chains
5. **DeFi Strategy Simulation**: Model different DeFi strategies and their potential outcomes
6. **Market Sentiment Analysis**: Incorporate news and social media sentiment into recommendations
7. **Automated Rebalancing**: Suggest specific trades to achieve target allocations

## Conclusion

You've built a powerful DeFi portfolio advisor with Alith that can analyze holdings, assess risk profiles, and generate personalized recommendations. This application demonstrates how AI agents can simplify complex financial analysis and provide valuable insights to cryptocurrency investors.

By combining on-chain data, financial analysis tools, and natural language processing, you've created an assistant that can help users make more informed decisions about their digital asset portfolios. 