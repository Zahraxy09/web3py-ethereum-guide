# Web3.py — Interacting with Ethereum Using Python

> A practical, code-first guide to Web3.py: connect to Ethereum, read blockchain data, send transactions, interact with smart contracts, and build real on-chain tools — all in Python.

---

## Table of Contents

1. [What Is Web3.py?](#what-is-web3py)
2. [How Python Connects to Ethereum](#how-python-connects-to-ethereum)
3. [Installation and Setup](#installation-and-setup)
4. [Connecting to a Node](#connecting-to-a-node)
5. [Reading Blockchain Data](#reading-blockchain-data)
6. [Working with Wallets and Addresses](#working-with-wallets-and-addresses)
7. [Sending Transactions](#sending-transactions)
8. [Interacting with Smart Contracts](#interacting-with-smart-contracts)
9. [Listening to Events](#listening-to-events)
10. [Working with ERC-20 Tokens](#working-with-erc-20-tokens)
11. [Building a Real Tool — Wallet Monitor](#building-a-real-tool--wallet-monitor)
12. [Common Errors and How to Fix Them](#common-errors-and-how-to-fix-them)
13. [Security Best Practices](#security-best-practices)
14. [Conclusion](#conclusion)

---

## What Is Web3.py?

**Web3.py** is a Python library that lets you interact with the Ethereum blockchain — read data, send transactions, deploy contracts, and call smart contract functions — all from Python code.

It is the Python equivalent of:
- **ethers.js** / **web3.js** in JavaScript
- **ethers-rs** in Rust

Any action you can do in MetaMask or Etherscan, you can automate with Web3.py.

### What You Can Build With It

| Use Case | Example |
|---|---|
| Wallet monitoring | Alert when an address receives ETH |
| DeFi automation | Auto-compound yield farming rewards |
| Token analytics | Track ERC-20 transfers in real time |
| Contract interaction | Call Uniswap, Aave, or your own contracts |
| On-chain data extraction | Build a price tracker or portfolio tool |
| Transaction bots | Arbitrage, liquidation, or sniping bots |
| Testing scripts | Verify contract behavior on testnet |

---

## How Python Connects to Ethereum

Python doesn't talk to Ethereum directly. It connects through a **node** — a server running Ethereum client software that has a copy of the blockchain and exposes an API.

```
Your Python Script
       ↓  HTTP / WebSocket / IPC
   Ethereum Node (RPC endpoint)
       ↓
   Ethereum Network
```

You have three options for your node:

| Option | How | Best For |
|---|---|---|
| **Hosted RPC** | Alchemy, Infura, QuickNode (free tier) | Getting started, low traffic |
| **Local node** | Run Geth or Erigon yourself | Full control, high traffic |
| **Local testnet** | Hardhat / Anvil (in-memory) | Testing and development |

For most projects, a hosted RPC provider is the fastest way to start.

---

## Installation and Setup

```bash
# Create a virtual environment (recommended)
python -m venv web3env
source web3env/bin/activate    # Linux/macOS
web3env\Scripts\activate       # Windows

# Install Web3.py
pip install web3

# Optional but useful
pip install python-dotenv      # For managing API keys in .env files
pip install requests           # For HTTP calls
```

### Verify Installation

```python
from web3 import Web3
print(Web3.api)   # Should print the Web3.py version
```

### Get a Free RPC Endpoint

1. Go to [alchemy.com](https://www.alchemy.com) and create a free account
2. Create a new app → Select **Ethereum** → **Mainnet** (or **Sepolia** for testnet)
3. Copy your **API key** / **HTTP URL**

It looks like:
```
https://eth-mainnet.g.alchemy.com/v2/your-api-key-here
```

Store it in a `.env` file — never hardcode it:

```bash
# .env
ALCHEMY_URL=https://eth-mainnet.g.alchemy.com/v2/your-api-key-here
PRIVATE_KEY=0xyour-private-key-here   # Only needed for sending transactions
```

Add `.env` to `.gitignore` immediately:
```bash
echo ".env" >> .gitignore
```

---

## Connecting to a Node

```python
import os
from web3 import Web3
from dotenv import load_dotenv

load_dotenv()

# Connect via HTTP (standard)
w3 = Web3(Web3.HTTPProvider(os.getenv("ALCHEMY_URL")))

# Check connection
if w3.is_connected():
    print("✅ Connected to Ethereum")
    print(f"Latest block: {w3.eth.block_number}")
else:
    print("❌ Connection failed")
```

### Connection Types

```python
# HTTP — standard, request/response
w3 = Web3(Web3.HTTPProvider("https://eth-mainnet.g.alchemy.com/v2/YOUR_KEY"))

# WebSocket — persistent connection, needed for real-time event listening
w3 = Web3(Web3.WebsocketProvider("wss://eth-mainnet.g.alchemy.com/v2/YOUR_KEY"))

# IPC — local node only, fastest
w3 = Web3(Web3.IPCProvider("/path/to/geth.ipc"))

# Local testnet (Hardhat / Anvil)
w3 = Web3(Web3.HTTPProvider("http://127.0.0.1:8545"))
```

---

## Reading Blockchain Data

### Block Information

```python
# Latest block number
block_number = w3.eth.block_number
print(f"Current block: {block_number}")

# Get a specific block
block = w3.eth.get_block(block_number)
print(f"Block hash:      {block['hash'].hex()}")
print(f"Timestamp:       {block['timestamp']}")
print(f"Transactions:    {len(block['transactions'])}")
print(f"Gas used:        {block['gasUsed']:,}")
print(f"Gas limit:       {block['gasLimit']:,}")
print(f"Miner:           {block['miner']}")

# Get block with full transaction objects
block_full = w3.eth.get_block(block_number, full_transactions=True)
for tx in block_full['transactions'][:3]:
    print(f"  TX: {tx['hash'].hex()[:20]}... | Value: {w3.from_wei(tx['value'], 'ether'):.4f} ETH")
```

### Transaction Details

```python
tx_hash = "0x5c504ed432cb51138bcf09aa5e8a410dd4a1e204ef84bfed1be16dfba1b22060"

tx = w3.eth.get_transaction(tx_hash)
print(f"From:      {tx['from']}")
print(f"To:        {tx['to']}")
print(f"Value:     {w3.from_wei(tx['value'], 'ether')} ETH")
print(f"Gas price: {w3.from_wei(tx['gasPrice'], 'gwei')} Gwei")
print(f"Nonce:     {tx['nonce']}")

# Transaction receipt (only available after mining)
receipt = w3.eth.get_transaction_receipt(tx_hash)
print(f"Status:    {'✅ Success' if receipt['status'] == 1 else '❌ Failed'}")
print(f"Gas used:  {receipt['gasUsed']:,}")
print(f"Block:     {receipt['blockNumber']}")
```

### Gas Price and Network State

```python
# Current gas price
gas_price = w3.eth.gas_price
print(f"Gas price: {w3.from_wei(gas_price, 'gwei'):.2f} Gwei")

# EIP-1559 fee data
fee_history = w3.eth.fee_history(10, 'latest', [25, 50, 75])
base_fee = fee_history['baseFeePerGas'][-1]
print(f"Base fee: {w3.from_wei(base_fee, 'gwei'):.2f} Gwei")

# Chain ID
print(f"Chain ID: {w3.eth.chain_id}")
# 1 = Ethereum Mainnet
# 11155111 = Sepolia Testnet
# 56 = BNB Smart Chain
```

---

## Working with Wallets and Addresses

### Address Utilities

```python
address = "0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045"  # vitalik.eth

# Check if valid Ethereum address
print(w3.is_address(address))   # True

# Convert to checksum address (standard format)
checksum = w3.to_checksum_address(address.lower())
print(checksum)

# Get ETH balance
balance_wei = w3.eth.get_balance(checksum)
balance_eth = w3.from_wei(balance_wei, 'ether')
print(f"Balance: {balance_eth:.4f} ETH")

# Get transaction count (nonce)
nonce = w3.eth.get_transaction_count(checksum)
print(f"Total transactions sent: {nonce}")
```

### Unit Conversions

```python
# ETH → Wei (smallest unit, 1 ETH = 10^18 Wei)
one_eth_in_wei = w3.to_wei(1, 'ether')
print(one_eth_in_wei)   # 1000000000000000000

# Wei → ETH
w3.from_wei(1000000000000000000, 'ether')   # Decimal('1')

# Gwei (used for gas prices)
w3.to_wei(20, 'gwei')       # 20000000000
w3.from_wei(20000000000, 'gwei')   # Decimal('20')

# Common units
# 1 ETH = 10^9 Gwei = 10^18 Wei
```

### Creating a New Wallet

```python
from eth_account import Account
import secrets

# Generate a random private key
private_key = "0x" + secrets.token_hex(32)
account = Account.from_key(private_key)

print(f"Address:     {account.address}")
print(f"Private key: {private_key}")

# ⚠️ NEVER share your private key
# ⚠️ Store it securely — whoever has it controls the wallet
```

### Signing Messages (Off-Chain)

```python
from eth_account.messages import encode_defunct

private_key = os.getenv("PRIVATE_KEY")
account = Account.from_key(private_key)

message = "I confirm ownership of this wallet."
msg = encode_defunct(text=message)
signed = account.sign_message(msg)

print(f"Signature: {signed.signature.hex()}")

# Recover signer from signature (verification)
recovered = Account.recover_message(msg, signature=signed.signature)
print(f"Signer: {recovered}")
print(f"Match:  {recovered == account.address}")
```

---

## Sending Transactions

### Sending ETH

```python
from web3 import Web3
from eth_account import Account
import os
from dotenv import load_dotenv

load_dotenv()

w3 = Web3(Web3.HTTPProvider(os.getenv("ALCHEMY_URL")))
account = Account.from_key(os.getenv("PRIVATE_KEY"))

recipient = "0xRecipientAddressHere"
amount_eth = 0.001   # Amount to send

# Build transaction
tx = {
    'nonce': w3.eth.get_transaction_count(account.address),
    'to': w3.to_checksum_address(recipient),
    'value': w3.to_wei(amount_eth, 'ether'),
    'gas': 21000,                                # Standard ETH transfer gas
    'maxFeePerGas': w3.to_wei(30, 'gwei'),       # EIP-1559: max total fee
    'maxPriorityFeePerGas': w3.to_wei(2, 'gwei'),# EIP-1559: tip to validator
    'chainId': w3.eth.chain_id,
    'type': '0x2',                               # EIP-1559 transaction type
}

# Sign
signed_tx = account.sign_transaction(tx)

# Broadcast
tx_hash = w3.eth.send_raw_transaction(signed_tx.rawTransaction)
print(f"Transaction sent: {tx_hash.hex()}")

# Wait for confirmation
receipt = w3.eth.wait_for_transaction_receipt(tx_hash, timeout=120)
print(f"Status: {'✅ Success' if receipt['status'] == 1 else '❌ Failed'}")
print(f"Gas used: {receipt['gasUsed']}")
print(f"View on Etherscan: https://sepolia.etherscan.io/tx/{tx_hash.hex()}")
```

### Estimating Gas Before Sending

```python
# Always estimate gas before sending — avoids out-of-gas failures
tx_params = {
    'from': account.address,
    'to': recipient,
    'value': w3.to_wei(0.001, 'ether'),
}

estimated_gas = w3.eth.estimate_gas(tx_params)
print(f"Estimated gas: {estimated_gas}")

# Add 20% buffer for safety
safe_gas = int(estimated_gas * 1.2)
```

---

## Interacting with Smart Contracts

### The ABI — Contract Interface

To call a contract, you need its **ABI** (Application Binary Interface) — a JSON description of its functions. You can find it on Etherscan under the "Contract" tab.

### Read-Only Contract Calls (Free)

```python
# Example: Read Chainlink ETH/USD price feed
CHAINLINK_ETH_USD = "0x5f4eC3Df9cbd43714FE2740f5E3616155c5b8419"

ABI = [
    {
        "inputs": [],
        "name": "latestRoundData",
        "outputs": [
            {"name": "roundId", "type": "uint80"},
            {"name": "answer", "type": "int256"},
            {"name": "startedAt", "type": "uint256"},
            {"name": "updatedAt", "type": "uint256"},
            {"name": "answeredInRound", "type": "uint80"}
        ],
        "stateMutability": "view",
        "type": "function"
    }
]

contract = w3.eth.contract(
    address=w3.to_checksum_address(CHAINLINK_ETH_USD),
    abi=ABI
)

# Call the function — no gas, no transaction
round_data = contract.functions.latestRoundData().call()
price = round_data[1] / 10**8   # Chainlink uses 8 decimals
print(f"ETH/USD price: ${price:,.2f}")
```

### State-Changing Contract Calls (Costs Gas)

```python
# Example: Call a custom contract's set() function
CONTRACT_ADDRESS = "0xYourContractAddress"

CONTRACT_ABI = [
    {
        "inputs": [{"name": "value", "type": "uint256"}],
        "name": "set",
        "outputs": [],
        "stateMutability": "nonpayable",
        "type": "function"
    },
    {
        "inputs": [],
        "name": "get",
        "outputs": [{"name": "", "type": "uint256"}],
        "stateMutability": "view",
        "type": "function"
    }
]

contract = w3.eth.contract(
    address=w3.to_checksum_address(CONTRACT_ADDRESS),
    abi=CONTRACT_ABI
)

# Read (free)
current_value = contract.functions.get().call()
print(f"Current value: {current_value}")

# Write (costs gas)
tx = contract.functions.set(42).build_transaction({
    'from': account.address,
    'nonce': w3.eth.get_transaction_count(account.address),
    'gas': 100000,
    'maxFeePerGas': w3.to_wei(30, 'gwei'),
    'maxPriorityFeePerGas': w3.to_wei(2, 'gwei'),
    'chainId': w3.eth.chain_id,
})

signed = account.sign_transaction(tx)
tx_hash = w3.eth.send_raw_transaction(signed.rawTransaction)
receipt = w3.eth.wait_for_transaction_receipt(tx_hash)
print(f"Value set! TX: {tx_hash.hex()}")
```

---

## Listening to Events

### Polling for Past Events

```python
# Get all Transfer events from USDC contract in last 100 blocks
USDC_ADDRESS = "0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48"

ERC20_ABI = [
    {
        "anonymous": False,
        "inputs": [
            {"indexed": True, "name": "from", "type": "address"},
            {"indexed": True, "name": "to", "type": "address"},
            {"indexed": False, "name": "value", "type": "uint256"}
        ],
        "name": "Transfer",
        "type": "event"
    }
]

usdc = w3.eth.contract(
    address=w3.to_checksum_address(USDC_ADDRESS),
    abi=ERC20_ABI
)

latest = w3.eth.block_number

# Get Transfer events from last 100 blocks
events = usdc.events.Transfer.get_logs(
    fromBlock=latest - 100,
    toBlock=latest
)

print(f"Found {len(events)} USDC transfers in last 100 blocks\n")
for event in events[:5]:
    value = event['args']['value'] / 10**6   # USDC has 6 decimals
    print(f"  From: {event['args']['from'][:10]}...")
    print(f"  To:   {event['args']['to'][:10]}...")
    print(f"  Amount: ${value:,.2f} USDC\n")
```

### Real-Time Event Watching (WebSocket)

```python
import asyncio
from web3 import AsyncWeb3

async def watch_transfers():
    w3 = AsyncWeb3(AsyncWeb3.WebSocketProvider(
        "wss://eth-mainnet.g.alchemy.com/v2/YOUR_KEY"
    ))

    usdc = w3.eth.contract(
        address=w3.to_checksum_address(USDC_ADDRESS),
        abi=ERC20_ABI
    )

    # Subscribe to Transfer events
    async for event in usdc.events.Transfer.subscribe():
        value = event['args']['value'] / 10**6
        if value > 1_000_000:   # Only large transfers > $1M
            print(f"🐋 Large transfer: ${value:,.0f} USDC")
            print(f"   From: {event['args']['from']}")
            print(f"   To:   {event['args']['to']}")

asyncio.run(watch_transfers())
```

---

## Working with ERC-20 Tokens

ERC-20 is the standard interface for fungible tokens. All major tokens (USDC, LINK, UNI, WETH) implement it.

```python
# Standard ERC-20 ABI (minimal)
ERC20_ABI = [
    {"inputs": [{"name": "account", "type": "address"}], "name": "balanceOf",
     "outputs": [{"name": "", "type": "uint256"}], "stateMutability": "view", "type": "function"},
    {"inputs": [], "name": "decimals",
     "outputs": [{"name": "", "type": "uint8"}], "stateMutability": "view", "type": "function"},
    {"inputs": [], "name": "symbol",
     "outputs": [{"name": "", "type": "string"}], "stateMutability": "view", "type": "function"},
    {"inputs": [], "name": "totalSupply",
     "outputs": [{"name": "", "type": "uint256"}], "stateMutability": "view", "type": "function"},
    {"inputs": [{"name": "to", "type": "address"}, {"name": "amount", "type": "uint256"}],
     "name": "transfer", "outputs": [{"name": "", "type": "bool"}],
     "stateMutability": "nonpayable", "type": "function"},
]

def get_token_info(token_address, wallet_address):
    token = w3.eth.contract(
        address=w3.to_checksum_address(token_address),
        abi=ERC20_ABI
    )

    symbol = token.functions.symbol().call()
    decimals = token.functions.decimals().call()
    total_supply_raw = token.functions.totalSupply().call()
    balance_raw = token.functions.balanceOf(
        w3.to_checksum_address(wallet_address)
    ).call()

    total_supply = total_supply_raw / 10**decimals
    balance = balance_raw / 10**decimals

    print(f"Token:        {symbol}")
    print(f"Total supply: {total_supply:,.2f} {symbol}")
    print(f"Your balance: {balance:,.6f} {symbol}")

    return balance

# Example: Check USDC balance
USDC = "0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48"
YOUR_WALLET = "0xYourWalletAddress"
get_token_info(USDC, YOUR_WALLET)
```

### Checking Multiple Token Balances

```python
TOKENS = {
    "USDC":  "0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48",
    "USDT":  "0xdAC17F958D2ee523a2206206994597C13D831ec7",
    "LINK":  "0x514910771AF9Ca656af840dff83E8264EcF986CA",
    "UNI":   "0x1f9840a85d5aF5bf1D1762F925BDADdC4201F984",
    "WETH":  "0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2",
}

wallet = "0xYourWalletAddress"

print(f"Portfolio for {wallet[:10]}...\n")
print(f"{'Token':<8} {'Balance':>20}")
print("-" * 30)

for symbol, address in TOKENS.items():
    token = w3.eth.contract(
        address=w3.to_checksum_address(address),
        abi=ERC20_ABI
    )
    decimals = token.functions.decimals().call()
    raw = token.functions.balanceOf(w3.to_checksum_address(wallet)).call()
    balance = raw / 10**decimals
    print(f"{symbol:<8} {balance:>20,.4f}")
```

---

## Building a Real Tool — Wallet Monitor

A complete script that monitors a wallet and alerts when it receives ETH or tokens:

```python
import time
import os
from web3 import Web3
from dotenv import load_dotenv

load_dotenv()

w3 = Web3(Web3.HTTPProvider(os.getenv("ALCHEMY_URL")))

WATCHED_WALLET = w3.to_checksum_address("0xWalletToWatch")
CHECK_INTERVAL = 15   # seconds

def get_eth_balance(address):
    return w3.from_wei(w3.eth.get_balance(address), 'ether')

def alert(message):
    print(f"\n🚨 ALERT: {message}")
    # Add your notification here:
    # - Send a Telegram message
    # - Send an email
    # - Call a webhook

def monitor():
    print(f"👀 Monitoring wallet: {WATCHED_WALLET}")
    print(f"   Checking every {CHECK_INTERVAL} seconds\n")

    last_balance = get_eth_balance(WATCHED_WALLET)
    last_block = w3.eth.block_number

    print(f"Starting ETH balance: {last_balance:.6f} ETH")
    print(f"Starting block: {last_block}\n")

    while True:
        try:
            current_balance = get_eth_balance(WATCHED_WALLET)
            current_block = w3.eth.block_number

            # Check ETH balance change
            if current_balance != last_balance:
                diff = current_balance - last_balance
                direction = "received" if diff > 0 else "sent"
                alert(
                    f"Wallet {direction} {abs(diff):.6f} ETH\n"
                    f"   New balance: {current_balance:.6f} ETH\n"
                    f"   Block: {current_block}"
                )
                last_balance = current_balance

            # Check new transactions involving wallet
            if current_block > last_block:
                for block_num in range(last_block + 1, current_block + 1):
                    block = w3.eth.get_block(block_num, full_transactions=True)
                    for tx in block['transactions']:
                        if tx['to'] == WATCHED_WALLET or tx['from'] == WATCHED_WALLET:
                            value_eth = w3.from_wei(tx['value'], 'ether')
                            direction = "IN ↙" if tx['to'] == WATCHED_WALLET else "OUT ↗"
                            print(f"  TX {direction} | {value_eth:.6f} ETH | {tx['hash'].hex()[:20]}...")

                last_block = current_block

            time.sleep(CHECK_INTERVAL)

        except Exception as e:
            print(f"Error: {e}")
            time.sleep(CHECK_INTERVAL)

if __name__ == "__main__":
    monitor()
```

---

## Common Errors and How to Fix Them

### `ValueError: {'code': -32000, 'message': 'insufficient funds'}`
```python
# Problem: Not enough ETH for gas
# Fix: Check balance before sending
balance = w3.eth.get_balance(account.address)
gas_cost = gas_limit * gas_price
if balance < (amount + gas_cost):
    print("Insufficient funds for transaction + gas")
```

### `web3.exceptions.ContractLogicError: execution reverted`
```python
# Problem: Smart contract rejected the transaction (require() failed)
# Fix: Call the function first to simulate and get the revert reason
try:
    contract.functions.someFunction(args).call({'from': account.address})
except Exception as e:
    print(f"Revert reason: {e}")
# Then fix the issue before sending the real transaction
```

### `nonce too low`
```python
# Problem: You're reusing a nonce that's already been used
# Fix: Always fetch nonce fresh before each transaction
nonce = w3.eth.get_transaction_count(account.address, 'pending')
# Use 'pending' to include unconfirmed transactions
```

### `replacement transaction underpriced`
```python
# Problem: Trying to replace a pending transaction with too low gas price
# Fix: New gas price must be at least 10% higher than the original
new_gas = int(original_gas * 1.15)  # 15% higher
```

### `timeout waiting for transaction receipt`
```python
# Problem: Transaction is pending too long (low gas price during congestion)
# Fix 1: Use a higher timeout
receipt = w3.eth.wait_for_transaction_receipt(tx_hash, timeout=300)

# Fix 2: Speed up by replacing transaction with higher gas (same nonce)
```

---

## Security Best Practices

### Never Hardcode Private Keys

```python
# ❌ NEVER DO THIS
private_key = "0x4c0883a69102937d6231471b5dbb6e538eba2ef8983e536d839f48"

# ✅ Always use environment variables
import os
from dotenv import load_dotenv
load_dotenv()
private_key = os.getenv("PRIVATE_KEY")
```

### Use Separate Wallets for Testing

```
Production wallet: holds real funds, used only in production code
Testing wallet:    holds only testnet funds, used for development
Hot wallet:        holds small operational amounts, never the main treasury
```

### Validate Addresses Before Sending

```python
def safe_send(to_address, amount_eth):
    # Validate address
    if not w3.is_address(to_address):
        raise ValueError(f"Invalid address: {to_address}")

    # Convert to checksum (catches typos in casing)
    to_address = w3.to_checksum_address(to_address)

    # Sanity check on amount
    if amount_eth <= 0:
        raise ValueError("Amount must be positive")

    if amount_eth > 1.0:
        confirm = input(f"Send {amount_eth} ETH? This is a large amount. Confirm (yes/no): ")
        if confirm.lower() != "yes":
            print("Cancelled.")
            return

    # Proceed with transaction...
```

### Simulate Before Sending

```python
# Always call() first to check if the transaction will succeed
try:
    result = contract.functions.riskyFunction(args).call({
        'from': account.address,
        'value': w3.to_wei(1, 'ether')
    })
    print(f"Simulation successful. Expected return: {result}")
except Exception as e:
    print(f"Transaction would fail: {e}")
    return   # Don't send if simulation fails
```

---

## Conclusion

Web3.py bridges Python's ecosystem — data science, automation, scripting — with the Ethereum blockchain. With it you can:

- **Read** any on-chain data: balances, transactions, contract state, events
- **Write** to the blockchain: send ETH, call contract functions, deploy contracts
- **Automate** workflows: monitor wallets, react to events, build trading bots
- **Integrate** blockchain into Python data pipelines and dashboards

**Key concepts from this article:**

- Web3.py connects to Ethereum through an RPC node (Alchemy, Infura, or local)
- All values are in Wei internally — use `to_wei()` and `from_wei()` to convert
- Read-only calls (`call()`) are free; state-changing calls require a signed transaction
- Always get the ABI from Etherscan to interact with existing contracts
- Use WebSocket connections for real-time event listening
- Store private keys in `.env` files — never in code or version control

The best next step: connect to Sepolia testnet, get test ETH from a faucet, and deploy the SimpleStorage contract from the [How Smart Contracts Work](../smart-contracts-deep-dive/README.md) article — then interact with it entirely through Web3.py.

---

## Further Reading

- [Web3.py Official Documentation](https://web3py.readthedocs.io/)
- [Alchemy Web3 Docs](https://docs.alchemy.com/reference/api-overview)
- [eth-account Documentation](https://eth-account.readthedocs.io/)
- [How Smart Contracts Work](../smart-contracts-deep-dive/README.md) ← Companion article
- [How to Use a Crypto Testnet](../crypto-testnet-guide/README.md) ← Practice safely

---

## License

Released under the [MIT License](LICENSE). Free to use, share, and adapt.

---

*Found this useful? Star the repo ⭐ and share it with a Python developer curious about blockchain.*
