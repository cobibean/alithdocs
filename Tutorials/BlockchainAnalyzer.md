# Building a Blockchain Data Analyzer with Alith

This tutorial guides you through creating an AI agent that can analyze on-chain data and provide insights. You'll build a tool that can examine transaction patterns, token flows, and smart contract interactions.

## What You'll Learn

- Fetching and processing blockchain data
- Creating specialized tools for data analysis
- Building an agent that can interpret complex data
- Generating insights from transaction patterns
- Visualizing blockchain data relationships

## Prerequisites

- Python 3.8+ installed
- Basic Python knowledge
- OpenAI or Anthropic API key
- Ethereum node access (via Infura, Alchemy, etc.)

## Step 1: Set Up Your Project

Create a new directory and set up a virtual environment:

```bash
mkdir blockchain-analyzer
cd blockchain-analyzer
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
```

Install the required dependencies:

```bash
pip install alith web3 pandas matplotlib python-dotenv
```

Create a `.env` file for your API keys:

```
OPENAI_API_KEY=your_openai_key_here
WEB3_PROVIDER_URI=https://mainnet.infura.io/v3/your_infura_key_here
```

## Step 2: Create the Basic Analyzer Structure

Create a file named `analyzer.py`:

```python
import os
import pandas as pd
import matplotlib.pyplot as plt
from dotenv import load_dotenv
from web3 import Web3
from alith import Agent

# Load environment variables
load_dotenv()

# Initialize Web3 connection
web3 = Web3(Web3.HTTPProvider(os.getenv('WEB3_PROVIDER_URI')))

# Check connection
if not web3.is_connected():
    raise Exception("Failed to connect to Ethereum node")

print(f"Connected to Ethereum network. Current block: {web3.eth.block_number}")

# Create tools for the agent
def get_transaction_history(params):
    """Tool to get transaction history for an address"""
    address = params.get('address')
    limit = params.get('limit', 10)
    
    try:
        # Validate address
        if not web3.is_address(address):
            return f"Invalid Ethereum address: {address}"
        
        # Convert to checksum address
        checksum_address = web3.to_checksum_address(address)
        
        # Get latest block number
        latest_block = web3.eth.block_number
        
        # List to store transactions
        transactions = []
        
        # Fetch the last 'limit' blocks
        for i in range(limit):
            if latest_block - i < 0:
                break
                
            block = web3.eth.get_block(latest_block - i, full_transactions=True)
            
            # Filter transactions by address
            for tx in block.transactions:
                if tx['from'] == checksum_address or tx['to'] == checksum_address:
                    tx_data = {
                        'hash': tx['hash'].hex(),
                        'from': tx['from'],
                        'to': tx['to'],
                        'value': web3.from_wei(tx['value'], 'ether'),
                        'block': block.number,
                        'timestamp': block.timestamp
                    }
                    transactions.append(tx_data)
        
        if not transactions:
            return f"No transactions found for address {checksum_address} in the last {limit} blocks."
        
        # Format the results
        result = f"Found {len(transactions)} transactions for {checksum_address}:\n\n"
        for tx in transactions:
            result += f"Block {tx['block']}: {tx['from']} → {tx['to']}: {tx['value']:.4f} ETH\n"
            
        return result
    
    except Exception as e:
        return f"Error retrieving transaction history: {str(e)}"

# Create a tool to get ETH balance
def get_eth_balance(params):
    """Tool to get ETH balance for an address"""
    address = params.get('address')
    
    try:
        # Validate address
        if not web3.is_address(address):
            return f"Invalid Ethereum address: {address}"
        
        # Convert to checksum address
        checksum_address = web3.to_checksum_address(address)
        
        # Get balance
        balance_wei = web3.eth.get_balance(checksum_address)
        balance_eth = web3.from_wei(balance_wei, 'ether')
        
        return f"Balance of {checksum_address}: {balance_eth:.6f} ETH"
    
    except Exception as e:
        return f"Error retrieving balance: {str(e)}"

# Initialize the Alith agent
agent = Agent(
    name="Blockchain Analyzer",
    model="gpt-4",
    preamble="""You are a blockchain data analysis assistant. You help users understand 
    Ethereum blockchain data, interpret transaction patterns, and provide insights about 
    on-chain activity. Use the provided tools to fetch and analyze data.""",
    tools=[
        {
            "name": "get_transaction_history",
            "description": "Get transaction history for an Ethereum address",
            "parameters": {
                "type": "object",
                "properties": {
                    "address": {
                        "type": "string",
                        "description": "Ethereum address to check"
                    },
                    "limit": {
                        "type": "integer",
                        "description": "Number of recent blocks to search (default: 10)"
                    }
                },
                "required": ["address"]
            },
            "handler": get_transaction_history
        },
        {
            "name": "get_eth_balance",
            "description": "Get the ETH balance of an Ethereum address",
            "parameters": {
                "type": "object",
                "properties": {
                    "address": {
                        "type": "string",
                        "description": "Ethereum address to check"
                    }
                },
                "required": ["address"]
            },
            "handler": get_eth_balance
        }
    ]
)

# Create a simple command-line interface
def main():
    print("Blockchain Data Analyzer powered by Alith")
    print("Enter 'exit' to quit")
    
    while True:
        user_input = input("\nEnter your query: ")
        
        if user_input.lower() == 'exit':
            break
        
        try:
            response = agent.prompt(user_input)
            print("\nAnalysis:")
            print(response)
        except Exception as e:
            print(f"Error: {str(e)}")

if __name__ == "__main__":
    main()
```

## Step 3: Add Token Analysis Tools

Enhance your analyzer with token tracking capabilities:

```python
# Add this import
from web3.contract import Contract

# Add this function for ERC20 interaction
def get_erc20_contract(token_address):
    """Get an ERC20 token contract instance"""
    # Standard ERC20 ABI for basic functions
    ERC20_ABI = [
        {"constant": True, "inputs": [], "name": "name", "outputs": [{"name": "", "type": "string"}], "payable": False, "stateMutability": "view", "type": "function"},
        {"constant": True, "inputs": [], "name": "symbol", "outputs": [{"name": "", "type": "string"}], "payable": False, "stateMutability": "view", "type": "function"},
        {"constant": True, "inputs": [], "name": "decimals", "outputs": [{"name": "", "type": "uint8"}], "payable": False, "stateMutability": "view", "type": "function"},
        {"constant": True, "inputs": [{"name": "owner", "type": "address"}], "name": "balanceOf", "outputs": [{"name": "", "type": "uint256"}], "payable": False, "stateMutability": "view", "type": "function"},
        {"constant": True, "inputs": [], "name": "totalSupply", "outputs": [{"name": "", "type": "uint256"}], "payable": False, "stateMutability": "view", "type": "function"}
    ]
    
    checksum_address = web3.to_checksum_address(token_address)
    contract = web3.eth.contract(address=checksum_address, abi=ERC20_ABI)
    return contract

# Create a tool to check ERC20 token balance
def get_token_balance(params):
    """Tool to get ERC20 token balance for an address"""
    address = params.get('address')
    token_address = params.get('token_address')
    
    try:
        # Validate addresses
        if not web3.is_address(address) or not web3.is_address(token_address):
            return "Invalid Ethereum address"
        
        # Convert to checksum addresses
        checksum_address = web3.to_checksum_address(address)
        checksum_token = web3.to_checksum_address(token_address)
        
        # Get token contract
        token_contract = get_erc20_contract(checksum_token)
        
        # Get token metadata
        try:
            token_symbol = token_contract.functions.symbol().call()
            token_decimals = token_contract.functions.decimals().call()
        except Exception:
            token_symbol = "UNKNOWN"
            token_decimals = 18
        
        # Get token balance
        balance_raw = token_contract.functions.balanceOf(checksum_address).call()
        balance = balance_raw / (10 ** token_decimals)
        
        return f"Token Balance for {checksum_address}: {balance:.6f} {token_symbol} (Token Contract: {checksum_token})"
    
    except Exception as e:
        return f"Error retrieving token balance: {str(e)}"
```

Add this to your agent's tools list:

```python
{
    "name": "get_token_balance",
    "description": "Get ERC20 token balance for an Ethereum address",
    "parameters": {
        "type": "object",
        "properties": {
            "address": {
                "type": "string",
                "description": "Ethereum address to check"
            },
            "token_address": {
                "type": "string",
                "description": "ERC20 token contract address"
            }
        },
        "required": ["address", "token_address"]
    },
    "handler": get_token_balance
}
```

## Step 4: Add Visualization Capabilities

Let's add a tool to visualize transaction data:

```python
import matplotlib
matplotlib.use('Agg')  # Use non-interactive backend

def visualize_transactions(params):
    """Create a visualization of transaction data"""
    address = params.get('address')
    days = params.get('days', 7)
    
    try:
        # Validate address
        if not web3.is_address(address):
            return f"Invalid Ethereum address: {address}"
        
        # Convert to checksum address
        checksum_address = web3.to_checksum_address(address)
        
        # Get latest block number
        latest_block = web3.eth.block_number
        
        # Calculate block number from 'days' ago (approximate)
        blocks_per_day = 7200  # ~15 sec per block
        start_block = max(0, latest_block - (days * blocks_per_day))
        
        # Create a DataFrame to hold transaction data
        df = pd.DataFrame(columns=['timestamp', 'block', 'type', 'value'])
        
        # Process blocks
        current_block = latest_block
        while current_block > start_block and len(df) < 100:  # Limit to 100 transactions
            try:
                block = web3.eth.get_block(current_block, full_transactions=True)
                
                # Process transactions in this block
                for tx in block.transactions:
                    if tx['from'] == checksum_address:
                        # Outgoing transaction
                        df = df.append({
                            'timestamp': block.timestamp,
                            'block': block.number,
                            'type': 'out',
                            'value': web3.from_wei(tx['value'], 'ether')
                        }, ignore_index=True)
                    elif tx['to'] == checksum_address:
                        # Incoming transaction
                        df = df.append({
                            'timestamp': block.timestamp,
                            'block': block.number,
                            'type': 'in',
                            'value': web3.from_wei(tx['value'], 'ether')
                        }, ignore_index=True)
            except Exception as e:
                print(f"Error processing block {current_block}: {e}")
            
            current_block -= 1
        
        if df.empty:
            return f"No transactions found for {checksum_address} in the specified time range."
        
        # Convert timestamp to datetime
        df['date'] = pd.to_datetime(df['timestamp'], unit='s')
        
        # Create the visualization
        plt.figure(figsize=(10, 6))
        
        # Plot outgoing transactions in red
        outgoing = df[df['type'] == 'out']
        plt.scatter(outgoing['date'], outgoing['value'], color='red', label='Outgoing', alpha=0.7)
        
        # Plot incoming transactions in green
        incoming = df[df['type'] == 'in']
        plt.scatter(incoming['date'], incoming['value'], color='green', label='Incoming', alpha=0.7)
        
        plt.title(f"Transaction History for {checksum_address[:10]}...{checksum_address[-8:]}")
        plt.xlabel("Date")
        plt.ylabel("Value (ETH)")
        plt.legend()
        plt.grid(True, alpha=0.3)
        
        # Save the figure
        filename = f"transactions_{checksum_address[:10]}.png"
        plt.savefig(filename)
        plt.close()
        
        return f"Visualization created and saved as {filename}. Found {len(df)} transactions: {len(incoming)} incoming and {len(outgoing)} outgoing."
    
    except Exception as e:
        return f"Error creating visualization: {str(e)}"
```

Add this to your agent's tools list:

```python
{
    "name": "visualize_transactions",
    "description": "Create a visualization of transaction data",
    "parameters": {
        "type": "object",
        "properties": {
            "address": {
                "type": "string",
                "description": "Ethereum address to analyze"
            },
            "days": {
                "type": "integer",
                "description": "Number of days to look back (default: 7)"
            }
        },
        "required": ["address"]
    },
    "handler": visualize_transactions
}
```

## Step 5: Add Smart Contract Analysis

Let's add a tool to analyze smart contract functions:

```python
def analyze_contract(params):
    """Analyze a smart contract's functions and activities"""
    contract_address = params.get('contract_address')
    
    try:
        # Validate address
        if not web3.is_address(contract_address):
            return f"Invalid Ethereum address: {contract_address}"
        
        # Convert to checksum address
        checksum_address = web3.to_checksum_address(contract_address)
        
        # Get contract code
        code = web3.eth.get_code(checksum_address).hex()
        
        if code == '0x':
            return f"No contract found at {checksum_address}. This is likely a regular wallet address."
        
        # Get contract bytecode size
        bytecode_size = len(code) / 2 - 1  # Subtract 1 for the '0x' prefix
        
        # Get contract transaction count
        tx_count = web3.eth.get_transaction_count(checksum_address)
        
        # Get contract balance
        balance = web3.eth.get_balance(checksum_address)
        balance_eth = web3.from_wei(balance, 'ether')
        
        # Try to determine if it's a token contract
        is_token = False
        token_info = ""
        
        try:
            token_contract = get_erc20_contract(checksum_address)
            symbol = token_contract.functions.symbol().call()
            name = token_contract.functions.name().call()
            total_supply_raw = token_contract.functions.totalSupply().call()
            decimals = token_contract.functions.decimals().call()
            total_supply = total_supply_raw / (10 ** decimals)
            
            is_token = True
            token_info = f"Token Information:\n- Name: {name}\n- Symbol: {symbol}\n- Decimals: {decimals}\n- Total Supply: {total_supply}"
        except Exception:
            pass
        
        result = f"Contract Analysis for {checksum_address}:\n\n"
        result += f"Bytecode Size: {bytecode_size} bytes\n"
        result += f"Transaction Count: {tx_count}\n"
        result += f"Balance: {balance_eth} ETH\n\n"
        
        if is_token:
            result += token_info
        else:
            result += "This does not appear to be a standard ERC20 token contract."
        
        return result
    
    except Exception as e:
        return f"Error analyzing contract: {str(e)}"
```

Add this to your agent's tools list:

```python
{
    "name": "analyze_contract",
    "description": "Analyze a smart contract's functions and activities",
    "parameters": {
        "type": "object",
        "properties": {
            "contract_address": {
                "type": "string",
                "description": "Ethereum contract address to analyze"
            }
        },
        "required": ["contract_address"]
    },
    "handler": analyze_contract
}
```

## Step 6: Run and Use the Analyzer

Run your blockchain analyzer:

```bash
python analyzer.py
```

You can now analyze blockchain data with natural language queries:

- "What's the ETH balance of vitalik.eth?"
- "Show me the transaction history for 0x1234...5678"
- "Analyze the contract at 0xabcd...ef00"
- "What ERC20 tokens does this address hold: 0x1234...5678"
- "Create a visualization of transactions for vitalik.eth over the last 14 days"

## Step 7: Extending Your Analyzer

Here are some ways to extend your blockchain analyzer:

1. **Multi-chain Support**: Add support for other blockchains like Polygon, Solana, or Binance Smart Chain
2. **Advanced Analytics**: Implement more sophisticated data analysis algorithms
3. **DeFi Integration**: Add tools to analyze DeFi positions, liquidity pools, and yield farming
4. **NFT Analysis**: Create tools for analyzing NFT collections and ownership
5. **Gas Optimization**: Add recommendations for gas optimization
6. **Security Analysis**: Implement tools to check for common security vulnerabilities in contracts
7. **Web Interface**: Create a web interface for easier interaction

## Conclusion

You've built a powerful blockchain data analyzer with Alith that can fetch and interpret on-chain data. This tool can be the foundation for more complex blockchain analytics applications, trading bots, or monitoring systems.

The combination of AI and blockchain enables new ways to understand and interact with on-chain data, providing insights that would be difficult to discover manually. 