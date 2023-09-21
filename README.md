# Description

This repository contains python examples of interaction with the DeFunds API. These scripts are designed to manage DeFunds automatically without making manual trades via a GUI.

 - [Supported Operations](#supported-operations)
 - [Installation](#installation)
 - [Usage](#usage)
  - [0x](#0x)
  - [GMX](#gmx)
    - [Increase Position](#increase-position)
    - [Decrease Position](#decrease-position)
 - [Error codes](#error-codes)

## Supported operations

Currently, DeFunds API supports these operations:
- Token swap via [0x](https://0x.org/docs/)
- [GMX](https://gmxio.gitbook.io/gmx/)
  - Increase Position
  - Decrease Position


## Installation

Set up your environment by creating a .env file in the root directory of the scripts and populate it with:
```dotenv
HOSTNAME=https://app.defunds.co/api
FUND_ID=your_fund_id
PRIVATE_KEY=your_private_key
```
Replace ‘your_fund_id’ and ‘your_private_key’ with your actual values.

## Usage

### 0x

To receive more detailed information about parameters, please visit [main 0x documentation](https://0x.org/docs/).

Supported ‘chainId’ values:

-   56 (Binance Smart Chain)
-   42161 (Arbitrum)
-   10 (Optimism)
-   420 (Optimism Goerli)

```python
tokenA = "0xFd086bC7CD5C481DCC9C85ebE478A1C0b69FCbb9"
tokenB = "0x2f2a2543b76a4166549f7aab2e75bef0aefc5b0f"
gas_limit = 1000000
gas_price_gwei = 0.1
gas_price = Web3.to_wei(gas_price_gwei, 'gwei')
node_url = "https://arb1.arbitrum.io/rpc" 
endpoint = os.environ.get("HOSTNAME")
fundId = os.environ.get("FUND_ID")
private_key = os.environ.get("PRIVATE_KEY")

w3 = Web3(Web3.HTTPProvider(node_url))
account: LocalAccount = Account.from_key(private_key)
w3.middleware_onion.add(construct_sign_and_send_raw_middleware(account))
abi_file_path = "trading.json"
with open(abi_file_path, "r") as abi_file:
    TradeABI = json.load(abi_file)
amount = "5000000000000000000"  # 5 USDT
url = f"{endpoint}/public/fund/{fundId}/swapTx?tokenA={tokenA}&tokenB={tokenB}&amount={amount}"
response = requests.get(url)
swapData = response.json()
url = f"{endpoint}/public/fund/{fundId}"
response = requests.get(url)
fundInfo = response.json()
trade_contract = w3.eth.contract(address=fundInfo["tradingAddress"], abi=TradeABI)
nonce = w3.eth.get_transaction_count(account.address)

swap_txn = trade_contract.functions.swap(
    Web3.to_checksum_address(tokenA),
    Web3.to_checksum_address(tokenB),
    Web3.to_int(text=amount),
    Web3.to_hex(hexstr=swapData["payload"])
).build_transaction({
    'chainId': 420,
    'gas': gas_limit,
    'gasPrice': gas_price,
    'nonce': nonce
})
signed_tx = w3.eth.account.sign_transaction(swap_txn, private_key=private_key)
tx_hash = w3.eth.send_raw_transaction(signed_tx.rawTransaction)
```

### GMX

To receive more detailed information about parameters, please visit [main GMX documentation](https://gmxio.gitbook.io/gmx/).

Supported chainId values:
-   42161 (Arbitrum)

#### Increase Position

```python
endpoint = os.environ.get("HOSTNAME")
fundId = os.environ.get("FUND_ID") 
private_key = os.environ.get("PRIVATE_KEY")
node_url = "https://arb1.arbitrum.io/rpc" 

w3 = Web3(Web3.HTTPProvider(node_url))
account: LocalAccount = Account.from_key(private_key)
w3.middleware_onion.add(construct_sign_and_send_raw_middleware(account))
abi_file_path = "trading.json"
with open(abi_file_path, "r") as abi_file:
    TradeABI = json.load(abi_file)

current_market_price = 27000 * 10**30 #price has 30 precision
slippage_percentage = 0.01
collateralToken = "0xFd086bC7CD5C481DCC9C85ebE478A1C0b69FCbb9"
indexToken = "0x2f2a2543b76a4166549f7aab2e75bef0aefc5b0f" 
collateralAmount = 10000000,
usdDelta = 3 * 10**31 
isLong = True
acceptablePrice =  int(current_market_price * (1+slippage_percentage) * 10**30)
executionFee = int(0.00018 * 10**18)
url = endpoint + "/public/fund/" + fundId
fundInfo = json.loads(response.text)
fundInfo = response.json()
trade_contract = w3.eth.contract(address=fundInfo["tradingAddress"], abi=TradeABI)
gas_limit = 10000000  
gas_price_gwei = 0.1  
gas_price = Web3.to_wei(gas_price_gwei, 'gwei')
execution_fee = 0.00018
gmx_txn = trade_contract.functions.gmxIncreasePosition( 	
	Web3.to_checksum_address(collateralToken), 	
	Web3.to_checksum_address(indexToken), 
	collateralAmount, 
	usdDelta, 
	isLong, 
	acceptablePrice, 
	executionFee
).build_transaction({ 
	'chainId': 42161, 
	'gas': gas_limit, 
	'gasPrice': gas_price, 
	'nonce': nonce,
	'value': Web3.to_wei(execution_fee, 'ether'), 
})
signed_tx = w3.eth.account.sign_transaction(swap_txn, private_key=private_key)
tx_hash = w3.eth.send_raw_transaction(signed_tx.rawTransaction)
```

#### Decrease Position
```python
endpoint = os.environ.get("HOSTNAME")
fundId = os.environ.get("FUND_ID")
private_key = os.environ.get("PRIVATE_KEY")
node_url = "https://arb1.arbitrum.io/rpc" 

w3 = Web3(Web3.HTTPProvider(node_url))
account: LocalAccount = Account.from_key(private_key)
w3.middleware_onion.add(construct_sign_and_send_raw_middleware(account))
abi_file_path = "trading.json"
with open(abi_file_path, "r") as abi_file:
    TradeABI = json.load(abi_file)
current_market_price = 27000 * 10**30 #price has 30 precision
slippage_percentage = 0.01
tokenFrom = "0x2f2a2543b76a4166549f7aab2e75bef0aefc5b0f"
indexToken = "0x2f2a2543b76a4166549f7aab2e75bef0aefc5b0f" 
tokenFromAmount = "0xFd086bC7CD5C481DCC9C85ebE478A1C0b69FCbb9",
collateralDelta = 0
usdDelta = 3 * 10**31 
isLong = True
acceptablePrice =  int(current_market_price * (1+slippage_percentage) * 10**30)
executionFee = int(minExecutionFee * 10**18)
url = endpoint + "/public/fund/" + fundId
fundInfo = json.loads(response.text)
fundInfo = response.json()
trade_contract = w3.eth.contract(address=fundInfo["tradingAddress"], abi=TradeABI)
gas_limit = 18000000  
gas_price_gwei = 0.1  
gas_price = Web3.to_wei(gas_price_gwei, 'gwei')  
  
execution_fee = 0.00018
gmx_txn = trade_contract.functions.gmxIncreasePosition( 	
	Web3.to_checksum_address(collateralToken), 	
	Web3.to_checksum_address(indexToken), 
	collateralAmount, 
	usdDelta, 
	isLong, 
	acceptablePrice, 
	executionFee
).build_transaction({ 
	'chainId': 42161, 
	'gas': gas_limit, 
	'gasPrice': gas_price, 
	'nonce': nonce,
	'value': Web3.to_wei(execution_fee, 'ether'), 
})
signed_tx = w3.eth.account.sign_transaction(swap_txn, private_key=private_key)
tx_hash = w3.eth.send_raw_transaction(signed_tx.rawTransaction)
```

## Error codes

TBD
