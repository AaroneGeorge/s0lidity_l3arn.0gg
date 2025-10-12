# 🧱 Foundry Quick Commands

## 🔹 Anvil

Run a local Ethereum development chain:

```bash
anvil
```

---

## 🔹 Forge

### Deploy a Contract

```bash
forge create SimpleStorage --rpc-url http://127.0.0.1:8545 --interactive --broadcast
```

* **[http://127.0.0.1:8545](http://127.0.0.1:8545)** → Anvil local chain URL
* **--interactive** → Prompts for your private key
* **--broadcast** → Required for deployment and on-chain transactions

---

### Run a Deployment Script (Local Chain)

```bash
forge script script/DeploySimpleStorage.s.sol
```

> Runs on local Anvil chain by default if `--rpc-url` isn’t specified.

### Run a Deployment Script (Custom RPC)

```bash
forge script script/DeploySimpleStorage.s.sol --rpc-url <RPC_URL> --private-key <PRIVATE_KEY> --broadcast
```

> Proper command for deployment using a specified RPC and private key.

---

### Format Codebase

```bash
forge fmt
```

> Formats the codebase uniformly.

---

## 🔹 Cast

### Convert HEX ↔ Decimal

```bash
cast --to-base <hex> dec
```

> Converts a hexadecimal value to decimal.

---

### Import a Wallet

```bash
cast wallet import <WALLET_NAME> --private-key <PRIVATE_KEY>
```

or interactively:

```bash
cast wallet import <WALLET_NAME> --interactive
```

> Imports or sets a wallet for transactions.

---

### Deploy Using a Stored Wallet

```bash
forge script script/DeploySimpleStorage.s.sol:DeploySimpleStorage \
  --rpc-url <RPC_URL> \
  --keystore ~/.foundry/keystores/<WALLET_NAME> \
  --broadcast -vvv
```

> Deploys a script using a wallet stored via `cast wallet`.

---

### Interact with a Contract (Send Transaction)

```bash
cast send <CONTRACT_ADDRESS> "store(uint256)" 123 \
  --rpc-url http://127.0.0.1:8545 \
  --private-key <PRIVATE_KEY>
```

> Sends a value to the `store()` function of the deployed contract.

#### Example Output:

```bash
blockHash            0x78d5a93108cb220c4b152c3169943f7cccbfdf4dd258dff408188d979db1c83c
blockNumber          3
from                 0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266
gasUsed              43740
status               1 (success)
transactionHash      0xa5ea82df1cd41d122673db1e4a7f049ede2493f17fa826b098a8fcfe6264747c
to                   0xe7f1725E7734CE288F8367e1Bb143E90bb3F0512
```

---

### Read Data from Contract

```bash
cast call 0xe7f1725E7734CE288F8367e1Bb143E90bb3F0512 "retrieve()"
```

> Retrieves stored data from the `retrieve()` view function.
> **Output:**

```
0x000000000000000000000000000000000000000000000000000000000000007b
```

To decode:

```bash
cast --to-base 0x000000000000000000000000000000000000000000000000000000000000007b dec
```

**Result:**

```
123
```

