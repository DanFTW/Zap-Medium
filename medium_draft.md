# How a user can send a request to update a data feed

The user has a choice to add a tip to the available feeds to update it's value, they do this by calling the addTip function on the ZapOps address.

### ZapOps address

BSCTestnet [0xE4d20DDCbA0B11fE1e0A214e30f04A2000A02C0b](https://testnet.bscscan.com/address/0xE4d20DDCbA0B11fE1e0A214e30f04A2000A02C0b)

You can interface with this contract via web3 or [ethers](https://docs.ethers.io/v5/), for this example I will use [web3.py](https://web3py.readthedocs.io/en/stable/).

## Set up your environment

For this step, you will define your public and private BSC address and contract addresses for web3 so you can define contract objects and set up your account to sign transactions.

First lets import the libraries

```python
from web3 import Web3
from web3.middleware import geth_poa_middleware

testnet = "https://data-seed-prebsc-1-s1.binance.org:8545"

testnet = Web3.HTTPProvider(testnet)
w3 = Web3(testnet)

# inject the poa compatibility middleware to the innermost layer
w3.middleware_onion.inject(geth_poa_middleware, layer=0)

# confirm that the connection succeeded
print("The next line confirms that BSC is connected\n", w3.clientVersion, "\n\n")

From: str = Web3.toChecksumAddress("PUBLIC_ADDRESS")
private_key: str = "PRIVATE_KEY"

# All contracts are on BSC Testnet
ZapOps_address: str = Web3.toChecksumAddress("0xE4d20DDCbA0B11fE1e0A214e30f04A2000A02C0b")

account = w3.eth.account.privateKeyToAccount(private_key)
w3.eth.default_account = account.address

# Contract object
Zap = w3.eth.contract(
    abi=[ABI_DATA],
    address=ZapOps_address
)
# ...
```

... in this step we're connecting to the public BSC testnet node to make transactions, using the `geth_poa_middleware` web3 middleware to ensure we can interact with this testnet, and defining our personal accounts and contract objects.

The abis also need to be defined for the contract object. There are many ways to do so and so I will not go over this step in this tutorial.

Next lets define the transaction to add a tip.

## Building the transaction

```python
# ...
nonce = w3.eth.get_transaction_count(From) + 2
tx = Zap.functions.addTip(
  1, 10  # requestID, tip amount
  ).buildTransaction(
    {
      "chainId": 97,
      "from": From,
      "nonce": nonce,
      "gas": int(gas),
    }
)

```
In this step we build the transaction before signing it.

The first operand of 1 is the requestID, the second operand of 10 is the amount of zap the user would like to tip to get their request updated.

The variable `gas` can be an estimated or fixed gas price, I gave an estimated gas using the `estimate_gas` web3 function

```python
# this needs to be defined above the previous code snippet if used
nonce = w3.eth.get_transaction_count(From) + 1
gas = w3.eth.estimate_gas({
      "chainId": 97,
      "from": From,
      "nonce": nonce,
    }, 'latest')
```

## Sign and send the Transaction

In this step we'll finally sign and send the transaction to the blockchain

```python
# ...
signed_tx = account.signTransaction(tx)
tx_hash = w3.toHex(w3.eth.sendRawTransaction(signed_tx.rawTransaction))
print("Tip Added at: ", tx_hash)
```

... after which, the tip for the given request is sent and a the miners on the oracle network will begin submitting an updated price for the given request.

**A mapping of which price feed belongs to a given request ID could be submitted along with this document once established**
