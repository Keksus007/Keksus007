
# DeFunds Scripts Documentation
This repository contains scripts that allow users to interact with the DeFunds platform, specifically for operations like "Swap", "GMX Increase Position", and "GMX Decrease Position". The scripts are designed to facilitate direct interactions without needing to manually process transactions via a GUI.
## Installation
### For the Swap Script, GMX Increase and GMX Decrease Scripts:
Set up your environment by creating a .env file in the root directory of the scripts and populate it with:
```python
HOSTNAME=https://app.defunds.co/api
FUND_ID=your_fund_id
PRIVATE_KEY=your_private_key
```
Replace ‘your_fund_id’ and ‘your_private_key’ with your actual values.
## Usage
### Swap script:
Supported ‘chainId’ values:

-   56 (Binance Smart Chain)
    
-   42161 (Arbitrum)
    
-   10 (Optimism)

===============================
#### 1. Configuration and Constants
===============================
```python
tokenA, tokenB = "0xFd086bC7CD5C481DCC9C85ebE478A1C0b69FCbb9", "0x82af49447d8a07e3bd95bd0d56f35241523fbab1"
gas_limit = 1000000
gas_price_gwei = 0.1
gas_price = Web3.to_wei(gas_price_gwei, 'gwei')
```
===============================
#### 2. Environment Variables and Checks
===============================
```python
endpoint = os.environ.get("HOSTNAME")
fundId = os.environ.get("FUND_ID")
if not fundId:
    print("Not fundId specified in the .env file")
    exit(1)

private_key = os.environ.get("PRIVATE_KEY")
```
===============================
#### 3. Web3 Setup and Middleware
===============================
```python
node_url = "https://arb1.arbitrum.io/rpc"  

w3 = Web3(Web3.HTTPProvider(node_url))
account: LocalAccount = Account.from_key(private_key)
w3.middleware_onion.add(construct_sign_and_send_raw_middleware(account))
```
===============================
#### 4. ABI Loading
===============================
```python
abi_file_path = "trading.json"
with open(abi_file_path, "r") as abi_file:
    TradeABI = json.load(abi_file)
```
===============================
#### 5. Transaction Data
===============================
```python
amount = "500000"  # 0.5 USDT
url = f"{endpoint}/public/fund/{fundId}/swapTx?tokenA={tokenA}&tokenB={tokenB}&amount={amount}"
response = requests.get(url)
swapData = response.json()
print(json.dumps(swapData, indent=4))
```
 ===============================
 #### 6. Fetch Fund Information
===============================
```python
url = f"{endpoint}/public/fund/{fundId}"
response = requests.get(url)
fundInfo = response.json()
```
 ===============================
 #### 7. Contract Interaction and Transaction Building
===============================
```python
trade_contract = w3.eth.contract(address=fundInfo["tradingAddress"], abi=TradeABI)
nonce = w3.eth.get_transaction_count(account.address)

swap_txn = trade_contract.functions.swap(
    Web3.to_checksum_address(swapData["swapper"]),
    Web3.to_checksum_address(tokenA),
    Web3.to_checksum_address(tokenB),
    Web3.to_int(text=amount),
    Web3.to_hex(hexstr=swapData["payload"])
).build_transaction({
    'chainId': 42161,
    'gas': gas_limit,
    'gasPrice': gas_price,
    'nonce': nonce
})
```
 ===============================
#### 8. Transaction Sending and Handling
 ===============================
```python
try:
    signed_tx = w3.eth.account.sign_transaction(swap_txn, private_key=private_key)
    tx_hash = w3.eth.send_raw_transaction(signed_tx.rawTransaction)
    print("Transaction hash:", tx_hash.hex())
except TransactionNotFound as e:
    print("Transaction reverted:", e)
```
### GMX Increase Position:
Supported chainId values:    
-   42161 (Arbitrum)
===============================
#### 1. Configuration and Environment Checks
===============================
```python
endpoint = os.environ.get("HOSTNAME")
fundId = os.environ.get("FUND_ID") 
if  not fundId: 
	print("Not fundId specified in the .env file") 
	exit(1)
	 
private_key = os.environ.get("PRIVATE_KEY")
```
===============================
#### 2. Web3 Setup and Middleware
===============================
```python
node_url = "https://arb1.arbitrum.io/rpc" 

w3 = Web3(Web3.HTTPProvider(node_url))
account: LocalAccount = Account.from_key(private_key)
w3.middleware_onion.add(construct_sign_and_send_raw_middleware(account))
```
===============================
#### 3. ABI Loading
===============================
```python
abi_file_path = "trading.json"
with open(abi_file_path, "r") as abi_file:
    TradeABI = json.load(abi_file)
```
===============================
#### 4. Fetch Market Price and Set Transaction Parameters
===============================
```python
current_market_price = get_price() # Note: Ensure the function 'get_price' is defined. 
slippage_percentage = 0.01
```
===============================
#### 5. Set GMX Transaction Data
===============================
```python
collateralToken, indexToken = "0xFd086bC7CD5C481DCC9C85ebE478A1C0b69FCbb9", "0x2f2a2543b76a4166549f7aab2e75bef0aefc5b0f" 
collateralAmount, usdDelta = 10000000, 3 * 10**31 
isLong, acceptablePrice, executionFee = True, int(current_market_price * (1+slippage_percentage) * 10**30), int(0.00018 * 10**18) 

url = "{0}/public/fund/{1}/gmx/increase-position?collateralToken={2}&indexToken={3}&isLong={4}".format(endpoint, fundId, collateralToken, indexToken, int(isLong)) 
response = requests.request("GET", url) 
gmxData = json.loads(response.text) 
if  "signature"  not  in gmxData: 
	print("Error: 'signature' key not found in GMX data response") 
	exit(1) 
data = bytes.fromhex(gmxData["signature"][2:])
```
 ===============================
 #### 6. Fetch Fund Information and Set Contract
===============================
```python
url = endpoint + "/public/fund/" + fundId
fundInfo = json.loads(response.text)
fundInfo = response.json()
trade_contract = w3.eth.contract(address=fundInfo["tradingAddress"], abi=TradeABI)
```
 ===============================
 #### 7. Transaction Building
===============================
```python
nonce, gas_limit, gas_price_gwei, amount_in_eth = w3.eth.get_transaction_count(account.address), 10000000, 0.1, 0.00018 
gas_price = Web3.to_wei(gas_price_gwei, 'gwei') 
gmx_txn = trade_contract.functions.gmxIncreasePosition( 	
	Web3.to_checksum_address(collateralToken), 	
	Web3.to_checksum_address(indexToken), 
	collateralAmount, 
	usdDelta, 
	isLong, 
	acceptablePrice, 
	executionFee, 
	data 
).build_transaction({ 
	'chainId': 42161, 
	'gas': gas_limit, 
	'gasPrice': gas_price, 
	'nonce': nonce,
	'value': Web3.to_wei(amount_in_eth, 'ether'), 
})
```
 ===============================
#### 8. Transaction Sending and Handling
 ===============================
```python
try:
    signed_tx = w3.eth.account.sign_transaction(swap_txn, private_key=private_key)
    tx_hash = w3.eth.send_raw_transaction(signed_tx.rawTransaction)
    print("Transaction hash:", tx_hash.hex())
except TransactionNotFound as e:
    print("Transaction reverted:", e)
```
    
### GMX Decrease Position:
Supported chainId values:    
-   42161 (Arbitrum)
===============================
#### 1. Configuration and Environment Checks
===============================
```python
endpoint = os.environ.get("HOSTNAME")
fundId = os.environ.get("FUND_ID") 
if  not fundId: 
	print("Not fundId specified in the .env file") 
	exit(1)
	 
private_key = os.environ.get("PRIVATE_KEY")
```
===============================
#### 2. Web3 Setup and Middleware
===============================
```python
node_url = "https://arb1.arbitrum.io/rpc" 

w3 = Web3(Web3.HTTPProvider(node_url))
account: LocalAccount = Account.from_key(private_key)
w3.middleware_onion.add(construct_sign_and_send_raw_middleware(account))
```
===============================
#### 3. ABI Loading
===============================
```python
abi_file_path = "trading.json"
with open(abi_file_path, "r") as abi_file:
    TradeABI = json.load(abi_file)
```
===============================
#### 4. Fetch Market Price and Set Transaction Parameters
===============================
```python
current_market_price = get_price() # Note: Ensure the function 'get_price' is defined. 
slippage_percentage = 0.01
```
===============================
#### 5. Set GMX Transaction Data
===============================
```python
tokenFrom, indexToken = "0x2f2a2543b76a4166549f7aab2e75bef0aefc5b0f", "0x2f2a2543b76a4166549f7aab2e75bef0aefc5b0f" 
tokenFromAmount, collateralDelta, usdDelta = "0xFd086bC7CD5C481DCC9C85ebE478A1C0b69FCbb9", 0, 3 * 10**31 
isLong, acceptablePrice, executionFee = True, int(current_market_price * (1+slippage_percentage) * 10**30), int(0.00018 * 10**18) 

url = "{0}/public/fund/{1}/gmx/increase-position?collateralToken={2}&indexToken={3}&isLong={4}".format(endpoint, fundId, collateralToken, indexToken, int(isLong)) 
response = requests.request("GET", url) 
gmxData = json.loads(response.text) 
if  "signature"  not  in gmxData: 
	print("Error: 'signature' key not found in GMX data response") 
	exit(1) 
data = bytes.fromhex(gmxData["signature"][2:])
```
 ===============================
 #### 6. Fetch Fund Information and Set Contract
===============================
```python
url = endpoint + "/public/fund/" + fundId
fundInfo = json.loads(response.text)
fundInfo = response.json()
trade_contract = w3.eth.contract(address=fundInfo["tradingAddress"], abi=TradeABI)
```
 ===============================
 #### 7. Transaction Building
===============================
```python
nonce, gas_limit, gas_price_gwei, amount_in_eth = w3.eth.get_transaction_count(account.address), 10000000, 0.1, 0.00018 
gas_price = Web3.to_wei(gas_price_gwei, 'gwei') 
gmx_txn = trade_contract.functions.gmxIncreasePosition( 	
	Web3.to_checksum_address(collateralToken), 	
	Web3.to_checksum_address(indexToken), 
	collateralAmount, 
	usdDelta, 
	isLong, 
	acceptablePrice, 
	executionFee, 
	data 
).build_transaction({ 
	'chainId': 42161, 
	'gas': gas_limit, 
	'gasPrice': gas_price, 
	'nonce': nonce,
	'value': Web3.to_wei(amount_in_eth, 'ether'), 
})
```
 ===============================
#### 8. Transaction Sending and Handling
 ===============================
```python
try:
    signed_tx = w3.eth.account.sign_transaction(swap_txn, private_key=private_key)
    tx_hash = w3.eth.send_raw_transaction(signed_tx.rawTransaction)
    print("Transaction hash:", tx_hash.hex())
except TransactionNotFound as e:
    print("Transaction reverted:", e)
```

