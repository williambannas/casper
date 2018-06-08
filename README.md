# Casper

[![](https://img.shields.io/badge/made%20by-Ethereum%20Foundation-blue.svg?style=flat-square)](http://ethereum.org)
[![Build Status](https://travis-ci.org/ethereum/casper.svg?branch=master)](https://travis-ci.org/ethereum/casper)
[![Casper](https://img.shields.io/badge/gitter-Casper-4AB495.svg)](https://gitter.im/ethereum/casper)
[![Casper scaling and protocol economics](https://img.shields.io/badge/gitter-Casper%20scaling%20and%20protocol%20economics-4AB495.svg)](https://gitter.im/ethereum/casper-scaling-and-protocol-economics)
[![standard-readme compliant](https://img.shields.io/badge/readme%20style-standard-brightgreen.svg?style=flat-square)](https://github.com/RichardLitt/standard-readme)

> Implements Casper FFG (the Friendly Finality Gadget), a Proof-of-Stake finality protocol that can be layered on any block proposal mechanism.

## Background

- Implements a [Casper FFG](https://arxiv.org/abs/1710.09437) smart contract, written in [Vyper](https://github.com/ethereum/vyper).
- See this [Casper the Friendly Finality Gadget](https://arxiv.org/abs/1710.09437) paper by Vitalik Buterin and Virgil Griffith introducing Casper FFG.
- [EIP-1011](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-1011.md):
specification of the Casper the Friendly Finality Gadget (FFG) PoW/PoS consensus model.

## Installation

For macOS, with [brew](https://brew.sh/) installed:

```bash
brew install pandoc # required for a python dependency
```

For all systems:

```bash
pip3 install -r requirements.txt
```

NOTE: pip3 is a version of pip using python version 3.
NOTE: we suggest using virtualenv to sandbox your setup.

## Usage

- [VALIDATOR_GUIDE.md](https://github.com/ethereum/casper/blob/master/VALIDATOR_GUIDE.md):
information about implementing a Casper FFG validator.

## Tests

```bash
pytest tests
```

## Version of Vyper to Compile Casper
We recommend using the vyper version installed as a dependency from requirements.txt. 
NOTE: The latest version of Vyper throws an error (line 16) when compiling the Casper contract due to a syntax change of "units".
NOTE: this not needed if steps above followed
```bash
git clone https://github.com/ethereum/vyper.git@248c723288e84899908048efff4c3e0b12f0b3dc
```
The guide on how to install Vyper can be found [here](https://github.com/ethereum/vyper#installation).


## How to Build and Deploy Casper Contract

In this example we will be using [eth-tester](https://github.com/ethereum/eth-tester).


### Step 1: Imports
```bash
import os
import rlp

from decimal import (
    Decimal,
)

import eth_tester

from eth_tester import (
    EthereumTester,
    PyEVMBackend
)
from web3.providers.eth_tester import (
    EthereumTesterProvider,
)

from web3 import (
    Web3,
)

from vyper import (
    compiler,
    utils as vyper_utils,
)

```

### Step 2: Raise the Gas Limit

The casper contract costs alot of gas for all its tasks. When you deploy the contract via a normal create transaction, make sure that the block gas limit is high enough. 


```bash
setattr(eth_tester.backends.pyevm.main, 'GENESIS_GAS_LIMIT', 10**9) ## it costs a lot of gas to deploy the casper contract, so set a high gas limit
```
### Step 3: Create an Instance of Web3 (eth-tester)

[Link to the web3.py documentation.](https://github.com/ethereum/web3.py)
```bash
base_tester = EthereumTester(PyEVMBackend())  ## instantiates the eth-test suite 
w3 = Web3(EthereumTesterProvider(base_tester)) ## Create an instance of the web3 object
w3.eth.setGasPriceStrategy(0)
```

### Step 4: Deploy Helper Contracts
NOTE: In addition to the two helper contracts `msg_hasher` and `purity_checker` you also need to deploy the `rlp_decoder` Contract. On a production chain, an `rlp_decoder` contract is already deployed and vyper’s standard internal library knows it’s address and gives vyper contracts access to some functionality at that address.
**When using a test chain, these helper contract must be deployed before the casper contract.**

NOTE: These helper contracts can be deployed via the below pre-signed txs. Instructions for compiling and deploying via other methods will soon be available.

#### Define Address & Transaction Hex
```bash  
VYPER_RLP_DECODER_TX_SENDER = "0x39ba083c30fCe59883775Fc729bBE1f9dE4DEe11"

VYPER_RLP_DECODER_TX_HEX = "0xf9035b808506fc23ac0083045f788080b903486103305660006109ac5260006109cc527f0100000000000000000000000000000000000000000000000000000000000000600035046109ec526000610a0c5260006109005260c06109ec51101515585760f86109ec51101561006e5760bf6109ec510336141558576001610a0c52610098565b60013560f76109ec51036020035260005160f66109ec510301361415585760f66109ec5103610a0c525b61022060016064818352015b36610a0c511015156100b557610291565b7f0100000000000000000000000000000000000000000000000000000000000000610a0c5135046109ec526109cc5160206109ac51026040015260016109ac51016109ac5260806109ec51101561013b5760016109cc5161044001526001610a0c516109cc5161046001376001610a0c5101610a0c5260216109cc51016109cc52610281565b60b86109ec5110156101d15760806109ec51036109cc51610440015260806109ec51036001610a0c51016109cc51610460013760816109ec5114156101ac5760807f01000000000000000000000000000000000000000000000000000000000000006001610a0c5101350410151558575b607f6109ec5103610a0c5101610a0c5260606109ec51036109cc51016109cc52610280565b60c06109ec51101561027d576001610a0c51013560b76109ec510360200352600051610a2c526038610a2c5110157f01000000000000000000000000000000000000000000000000000000000000006001610a0c5101350402155857610a2c516109cc516104400152610a2c5160b66109ec5103610a0c51016109cc516104600137610a2c5160b66109ec5103610a0c510101610a0c526020610a2c51016109cc51016109cc5261027f565bfe5b5b5b81516001018083528114156100a4575b5050601f6109ac511115155857602060206109ac5102016109005260206109005103610a0c5261022060016064818352015b6000610a0c5112156102d45761030a565b61090051610a0c516040015101610a0c51610900516104400301526020610a0c5103610a0c5281516001018083528114156102c3575b50506109cc516109005101610420526109cc5161090051016109005161044003f35b61000461033003610004600039610004610330036000f31b2d4f"
```
```bash
MSG_HASHER_TX_SENDER = "0xD7a3BD6C9eA32efF147d067f907AE6b22d436F91"

MSG_HASHER_TX_HEX = "0xf9016d808506fc23ac0083026a508080b9015a6101488061000e6000396101565660007f01000000000000000000000000000000000000000000000000000000000000006000350460f8811215610038576001915061003f565b60f6810391505b508060005b368312156100c8577f01000000000000000000000000000000000000000000000000000000000000008335048391506080811215610087576001840193506100c2565b60b881121561009d57607f8103840193506100c1565b60c08112156100c05760b68103600185013560b783036020035260005101840193505b5b5b50610044565b81810360388112156100f4578060c00160005380836001378060010160002060e052602060e0f3610143565b61010081121561010557600161011b565b6201000081121561011757600261011a565b60035b5b8160005280601f038160f701815382856020378282600101018120610140526020610140f350505b505050505b6000f31b2d4f"
```
```bash
PURITY_CHECKER_TX_SENDER = "0xeA0f0D55EE82Edf248eD648A9A8d213FBa8b5081"

PURITY_CHECKER_TX_HEX = "0xf90467808506fc23ac00830583c88080b904546104428061000e60003961045056600061033f537c0100000000000000000000000000000000000000000000000000000000600035047f80010000000000000000000000000000000000000030ffff1c0e00000000000060205263a1903eab8114156103f7573659905901600090523660048237600435608052506080513b806020015990590160009052818152602081019050905060a0526080513b600060a0516080513c6080513b8060200260200159905901600090528181526020810190509050610100526080513b806020026020015990590160009052818152602081019050905061016052600060005b602060a05103518212156103c957610100601f8360a051010351066020518160020a161561010a57fe5b80606013151561011e57607f811315610121565b60005b1561014f5780607f036101000a60018460a0510101510482602002610160510152605e8103830192506103b2565b60f18114801561015f5780610164565b60f282145b905080156101725780610177565b60f482145b9050156103aa5760028212151561019e5760606001830360200261010051015112156101a1565b60005b156101bc57607f6001830360200261010051015113156101bf565b60005b156101d157600282036102605261031e565b6004821215156101f057600360018303602002610100510151146101f3565b60005b1561020d57605a6002830360200261010051015114610210565b60005b1561022b57606060038303602002610100510151121561022e565b60005b1561024957607f60038303602002610100510151131561024c565b60005b1561025e57600482036102605261031d565b60028212151561027d57605a6001830360200261010051015114610280565b60005b1561029257600282036102605261031c565b6002821215156102b157609060018303602002610100510151146102b4565b60005b156102c657600282036102605261031b565b6002821215156102e65760806001830360200261010051015112156102e9565b60005b156103035760906001830360200261010051015112610306565b60005b1561031857600282036102605261031a565bfe5b5b5b5b5b604060405990590160009052600081526102605160200261016051015181602001528090502054156103555760016102a052610393565b60306102605160200261010051015114156103755760016102a052610392565b60606102605160200261010051015114156103915760016102a0525b5b5b6102a051151561039f57fe5b6001830192506103b1565b6001830192505b5b8082602002610100510152600182019150506100e0565b50506001604060405990590160009052600081526080518160200152809050205560016102e05260206102e0f35b63c23697a8811415610440573659905901600090523660048237600435608052506040604059905901600090526000815260805181602001528090502054610300526020610300f35b505b6000f31b2d4f"
```

#### Fund Senders of Contract With 0.1 Ether to Deploy TXs
```bash
w3.eth.sendTransaction({'to': VYPER_RLP_DECODER_TX_SENDER, 'value': 10**17})
w3.eth.sendTransaction({'to': PURITY_CHECKER_TX_SENDER, 'value': 10**17})
w3.eth.sendTransaction({'to': MSG_HASHER_TX_SENDER, 'value': 10**17})
```

#### Deploy Dependency Contracts
```bash
rlp_tx_hash = w3.eth.sendRawTransaction(VYPER_RLP_DECODER_TX_HEX)
rlp_receipt = w3.eth.getTransactionReceipt(rlp_tx_hash)
rlp_decoder_address = rlp_receipt.contractAddress

msg_tx_hash = w3.eth.sendRawTransaction(MSG_HASHER_TX_HEX)
msg_receipt = w3.eth.getTransactionReceipt(msg_tx_hash)
msg_hasher_address = msg_receipt.contractAddress

purity_tx_hash = w3.eth.sendRawTransaction(PURITY_CHECKER_TX_HEX)
purity_receipt = w3.eth.getTransactionReceipt(purity_tx_hash)
purity_checker_address = purity_receipt.contractAddress
```

### Step 5: Casper Parameters
[Detailed Explaination of Casper Parameters](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-1011.md#casper-contract-params)

```bash
EPOCH_LENGTH = 10
WARM_UP_PERIOD = 20
WITHDRAWAL_DELAY = 8
DYNASTY_LOGOUT_DELAY = 5
BASE_INTEREST_FACTOR = Decimal('0.02')
BASE_PENALTY_FACTOR = Decimal('0.002')
MIN_DEPOSIT_SIZE = 1000 * 10**18  # 1000 ether

casper_args = [EPOCH_LENGTH, WARM_UP_PERIOD, WITHDRAWAL_DELAY, DYNASTY_LOGOUT_DELAY, msg_hasher_address, purity_checker_address, BASE_INTEREST_FACTOR, BASE_PENALTY_FACTOR, MIN_DEPOSIT_SIZE]
```

### Step 6: Compiling and Deploying Casper
```bash

file = open(os.getcwd()+('/casper/contracts/simple_casper.v.py')) #this will change depending on where the casper contract is located relative to your current working directory
casper_code = file.read()

casper_bytecode = compiler.compile(casper_code) #using vyper
casper_abi = compiler.mk_full_signature(casper_code) #using vyper
base_sender = base_tester.get_accounts()[-1] #account created by eth-tester containing 1 million eth


Casper = w3.eth.contract(abi=casper_abi, bytecode=casper_bytecode) #cteates casper contract object
tx_hash = Casper.constructor().transact({'from': base_sender}) #deploy casper contract
tx_receipt = w3.eth.getTransactionReceipt(tx_hash)

casper_address = tx_receipt.contractAddress

w3.eth.sendTransaction({'to': casper_address, 'value': 10**21})

casper_contract = w3.eth.contract(address=casper_address, abi=casper_abi)
casper_contract.functions.init(*casper_args).transact()
```

### Step 7: Interacting with the Casper Contract

As an exampe that the deployment was successful, we can check EPOCH_LENGTH. If you didn't change the initial parameters above, the following command should return 10.

```bash
casper_contract.functions.EPOCH_LENGTH().call()
```

## Contribute

Feel free to ask questions in our [Gitter room](https://gitter.im/ethereum/casper) or open an [issue](https://github.com/ethereum/casper/issues) for feature requests or bug reports. Feel free to make a PR!

## License

[UNLICENSE](LICENSE)