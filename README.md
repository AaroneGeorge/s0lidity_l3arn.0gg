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
### Verify Contract using etherscan api
```bash
forge verify-contract <DEPLOYED_CONTRACT_ADDRESS> <CONTRACT_PATH>:<CONTRACT_NAME> --chain-id <CHAIN_ID> --etherscan-api-key $ETHERSCAN_API_KEY
```
>If your contract has constructor parameters, you must include them encoded:
```bash
forge verify-contract 0x1234abcd... src/MyToken.sol:MyToken --constructor-args $(cast abi-encode "constructor(address,uint256)" 0xabc123... 1000) --chain-id 11155111 --etherscan-api-key $ETHERSCAN_API_KEY
```
>If you deployed via a forge script, Foundry will automatically cache your constructor args, so you can use the shorthand:
```bash
forge verify-contract --compiler-version v0.8.24+commit.e0e8fxx --num-of-optimizations 200 <DEPLOYED CA> src/MyToken.sol:MyToken
```

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

> Sends a value to the `store()` function of the deployed contract:

```bash
cast send <CONTRACT_ADDRESS> "store(uint256)" 123 \
  --rpc-url http://127.0.0.1:8545 \
  --private-key <PRIVATE_KEY>
```

> Another way to send tx with already existing cast wallet:
```bash
cast send <CONTRACT_ADDRESS> "store(uint256)" 123 \
  --rpc-url http://127.0.0.1:8545 \
  --keystore ~/.foundry/keystores/<WALLET_NAME>
```


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


## ⚡ Foundry zkSync

### 🧩 Installation

```bash
curl -L https://raw.githubusercontent.com/matter-labs/foundry-zksync/main/install-foundry-zksync | bash
```

> Installs the **Foundry zkSync** toolchain.

---

### 🔁 Switching Between Versions

**Switch back to standard Foundry:**

```bash
foundryup
```

**Check version:**

```bash
forge --version
```

**Switch to zkSync-enabled Foundry:**

```bash
foundryup-zksync
```

---

### 🧱 Compile a Contract for zkSync

```bash
forge compile --zksync
```

> Compiles your contracts using the zkSync compiler.

---

### 🚀 Run a Local zkSync Node

```bash
anvil-zksync
```

> Starts a local zkSync-compatible Anvil instance.

> 💡 For more advanced setups, it’s recommended to use **Docker containers** and the **zkSync CLI** for full node management and environment control.

---

### 🧾 Deploy a Contract on zkSync

```bash
forge create src/SimpleStorage.sol:SimpleStorage \
  --rpc-url <ZKSYNC_RPC_URL> \
  --private-key <PRIVATE_KEY> \
  --zksync
```

> Deploys your contract directly to the zkSync network.

---

# Other important infos while dev-maxxing
## 1. --legacy flag

### 🧠 Background:

Ethereum introduced **EIP-1559** to improve gas estimation and fee burning.
With it, transactions now have two new fields:

* `maxFeePerGas`
* `maxPriorityFeePerGas`

However, some networks or RPCs (like older testnets, private dev chains, or L2s that haven’t fully adopted EIP-1559) may **not support these new fields**.

---

### ⚙️ What `--legacy` does

When you add:

```bash
forge script script/Deploy.s.sol --rpc-url $RPC_URL --broadcast --legacy
```

It tells Foundry to:

* Use **`gasPrice`** instead of `maxFeePerGas` and `maxPriorityFeePerGas`.
* Send **legacy-style transactions**, compatible with older networks or clients.

---

### 🧩 When to use `--legacy`

Use it if:

* You get an error like:

  ```
  unsupported transaction type: 2
  ```

  (Type 2 is EIP-1559 transactions.)
* You’re deploying to an older chain, local anvil node, or custom EVM that only supports **legacy (type 0)** transactions.
* Your RPC provider doesn’t recognize `maxFeePerGas`.

---

### ✅ Example:

```bash
forge script script/Deploy.s.sol:Deploy --rpc-url http://localhost:8545 \
  --broadcast --legacy --slow
```

This sends a **type 0 transaction** (legacy gas model).

`--slow` flag is used to send and confirm each transaction one at a time, waiting for each to be mined before sending the next.


## 2. The Tx types

### 🧱 1. **Type 0 – Legacy Transactions**

**Introduced:** Pre-EIP-1559 (the original Ethereum transaction format).
**Transaction type byte:** *not explicitly included* (i.e., old default).

### 🔹 Structure:

* `nonce`
* `gasPrice`
* `gasLimit`
* `to`
* `value`
* `data`
* `v, r, s` (signature)

### 🔹 Characteristics:

* Uses a **single gasPrice** field.
* User bids a total gas price; miners prioritize higher bids.
* **No fee burn**, all gas fee goes to the miner.

### 🔹 When you see this:

* When using `--legacy` in Foundry.
* On older chains (like early Ethereum or some EVM forks).

### 🔹 Example:

```json
{
  "gasPrice": "20000000000",
  "nonce": 1,
  "to": "0xabc...",
  "value": "0x0"
}
```

---

### ⚙️ 2. **Type 1 – Access List Transactions (EIP-2930)**

**Introduced:** April 2021 (EIP-2930, just before EIP-1559).
**Transaction type byte:** `0x01`.

### 🔹 Structure:

Same as Type 0, **plus**:

* `accessList`: a list of addresses and storage keys the transaction will touch.

### 🔹 Why:

* Helps clients prefetch state, reducing gas cost for certain use cases.
* Useful for contracts with predictable storage access patterns.

### 🔹 Characteristics:

* Still uses **`gasPrice`** (no EIP-1559 fields yet).
* Mostly used on L2s or for optimization.

### 🔹 Example:

```json
{
  "type": "0x1",
  "gasPrice": "20000000000",
  "accessList": [
    {
      "address": "0xabc...",
      "storageKeys": ["0x000...001"]
    }
  ]
}
```

---

### 🔥 3. **Type 2 – EIP-1559 Transactions**

**Introduced:** August 2021 (EIP-1559, London Hard Fork).
**Transaction type byte:** `0x02`.

### 🔹 Structure:

* `maxFeePerGas`
* `maxPriorityFeePerGas`
* `accessList`
* and the usual fields (nonce, to, data, etc.)

### 🔹 Characteristics:

* Introduces **base fee** (burned per block).
* **maxFeePerGas** = total cap willing to pay.
* **maxPriorityFeePerGas** = tip to the miner.
* Makes gas estimation more stable and predictable.

### 🔹 Example:

```json
{
  "type": "0x2",
  "maxFeePerGas": "40000000000",
  "maxPriorityFeePerGas": "2000000000",
  "accessList": [],
  "nonce": 1,
  "to": "0xabc...",
  "value": "0x0"
}
```

---

### ⚖️ Summary Table

| Type | EIP  | Name        | Fee Model                              | Access List | Common Fields             | Typical Use            |
| ---- | ---- | ----------- | -------------------------------------- | ----------- | ------------------------- | ---------------------- |
| 0    | –    | Legacy      | `gasPrice`                             | ❌           | nonce, gasPrice, gasLimit | Pre-EIP-1559, old RPCs |
| 1    | 2930 | Access List | `gasPrice`                             | ✅           | + accessList              | Optimization use cases |
| 2    | 1559 | Dynamic Fee | `maxFeePerGas`, `maxPriorityFeePerGas` | ✅           | + accessList              | Default since 2021     |

---

## 3. The `vm` functions or cheatcodes in foundry

### 🧠 What they are

In Foundry, `vm` functions are called ***cheatcodes***.

They are **special helper functions** provided by the Forge testing environment (not part of Solidity itself) that allow you to:

* Manipulate blockchain state
* Mock behavior
* Simulate different conditions
* Control execution during testing

---

### ⚙️ Major Categories of Cheatcodes (`vm` functions)

### 🕒 1. **Time & Block Manipulation**

| Cheatcode                | Description                                             |
| ------------------------ | ------------------------------------------------------- |
| `vm.warp(uint256)`       | Sets `block.timestamp`                                  |
| `vm.roll(uint256)`       | Sets `block.number`                                     |
| `vm.fee(uint256)`        | Sets `block.basefee`                                    |
| `vm.chainId(uint256)`    | Sets `block.chainid`                                    |
| `vm.prevrandao(uint256)` | Sets `block.prevrandao` (previously `block.difficulty`) |

---

### 💰 2. **Account & Balance Control**

| Cheatcode                   | Description                                    |
| --------------------------- | ---------------------------------------------- |
| `vm.deal(address, uint256)` | Sets ETH balance for an address                |
| `vm.prank(address)`         | Makes the next call come **from** that address |
| `vm.startPrank(address)`    | Starts persistent impersonation of an address  |
| `vm.stopPrank()`            | Stops impersonation                            |
| `vm.addr(uint256)`          | Derives an address from a private key          |
| `vm.sign(uint256, bytes32)` | Signs a message hash with a given private key  |

---

### 📄 3. **Storage & State**

| Cheatcode                                       | Description                         |
| ----------------------------------------------- | ----------------------------------- |
| `vm.store(address, bytes32 key, bytes32 value)` | Writes directly to contract storage |
| `vm.load(address, bytes32 key)`                 | Reads raw storage at a given slot   |
| `vm.snapshot()`                                 | Takes a snapshot of the EVM state   |
| `vm.revertTo(uint256 snapshotId)`               | Reverts to a previous snapshot      |

---

### ⚡ 4. **Error & Revert Testing**

| Cheatcode                       | Description                                    |
| ------------------------------- | ---------------------------------------------- |
| `vm.expectRevert()`             | Expects a revert in the next call              |
| `vm.expectRevert(bytes)`        | Expects a revert with specific error data      |
| `vm.expectEmit()`               | Expects an event to be emitted                 |
| `vm.expectCall(address, bytes)` | Expects a call to a specific contract/function |

---

### 🔐 5. **Environment & Configuration**

| Cheatcode               | Description                             |
| ----------------------- | --------------------------------------- |
| `vm.envString(string)`  | Reads an environment variable (string)  |
| `vm.envUint(string)`    | Reads an environment variable (uint)    |
| `vm.envAddress(string)` | Reads an environment variable (address) |
| `vm.envBool(string)`    | Reads an environment variable (bool)    |

---

### 🧪 6. **Mocking & Fuzzing Helpers**

| Cheatcode                              | Description                                    |
| -------------------------------------- | ---------------------------------------------- |
| `vm.assume(bool condition)`            | Sets a constraint for fuzzing tests            |
| `vm.bound(uint256, uint256, uint256)`  | Bounds a fuzzed value to a specific range      |
| `vm.label(address, string)`            | Labels an address for better stack traces      |
| `vm.record()` / `vm.accesses(address)` | Records all storage reads/writes for debugging |

---

### 🪄 7. **Miscellaneous / Advanced**

| Cheatcode                      | Description                                                                |
| ------------------------------ | -------------------------------------------------------------------------- |
| `vm.ffi(string[])`             | Runs a shell command and returns stdout (⚠️ powerful, disabled by default) |
| `vm.broadcast()`               | Used in scripts for broadcasting transactions                              |
| `vm.serialize*()`              | Used to build structured JSON data for debugging/reporting                 |
| `vm.getCode(string)`           | Loads bytecode from a file                                                 |
| `vm.setNonce(address, uint64)` | Sets the transaction nonce of an account                                   |

---
## 4. AAA testing framework

It stands for:

> **Arrange → Act → Assert**

Each test follows this structure:

1. **Arrange** → Set up your environment and inputs
2. **Act** → Execute the function or transaction being tested
3. **Assert** → Verify that the result is what you expect

---

### 🧩 Example (Solidity + Foundry)

Let’s say you’re testing a simple ERC20 transfer.

```solidity
import "forge-std/Test.sol";
import "../src/MyToken.sol";

contract MyTokenTest is Test {
    MyToken token;
    address alice = address(1);
    address bob = address(2);

    function setUp() public {
        token = new MyToken("TestToken", "TT", 18);
        token.mint(alice, 100 ether);
    }

    function testTransfer() public {
        /*** ARRANGE ***/
        vm.startPrank(alice);
        uint256 initialBalance = token.balanceOf(bob);

        /*** ACT ***/
        token.transfer(bob, 50 ether);

        /*** ASSERT ***/
        uint256 finalBalance = token.balanceOf(bob);
        assertEq(finalBalance, initialBalance + 50 ether, "Bob should receive 50 tokens");

        vm.stopPrank();
    }
}
```

---

### 🧪 How it maps in Solidity testing

| Phase       | Example operations                                                                   |
| ----------- | ------------------------------------------------------------------------------------ |
| **Arrange** | Deploy contracts, set balances, assign roles, mock data, warp time, etc.             |
| **Act**     | Call the function or emit transaction you’re testing                                 |
| **Assert**  | Use `assertEq`, `assertTrue`, `expectRevert`, `expectEmit`, etc., to verify outcomes |

---

### 🧱 Example: Testing a revert condition

```solidity
function testCannotTransferMoreThanBalance() public {
    /*** ARRANGE ***/
    vm.startPrank(alice);
    uint256 aliceBalance = token.balanceOf(alice);

    /*** ACT ***/
    vm.expectRevert("ERC20: transfer amount exceeds balance");
    token.transfer(bob, aliceBalance + 1);

    /*** ASSERT ***/
    // No balance change should occur
    assertEq(token.balanceOf(alice), aliceBalance);
    vm.stopPrank();
}
```

## 5. Types of tests

### 🧱 Folder Structure Overview

```
test/
├── unit/          # Isolated tests for single contracts or functions
├── integration/   # Tests interactions between multiple contracts
├── fuzz/          # Randomized property-based tests
├── invariant/     # Long-running tests ensuring system-level properties always hold
├── fork/          # Tests using real mainnet/testnet state (via fork)
└── mocks/         # Mock contracts for simulation or dependency isolation
```

---

### 🧪 1. **Unit Testing**

**Goal:**
Test individual **functions or contracts** in isolation — no external dependency.
Used to verify *smallest logical units* of code.

**Example:**

* Test that `deposit()` updates balances correctly.
* Check that `transfer()` emits the right event.

---

### 🔗 2. **Integration Testing**

**Goal:**
Test **interactions** between multiple contracts or modules together.
Ensures they work properly as a system.

**Example:**

* Token + Vault integration (depositing ERC20 tokens).
* LendingPool interacting with PriceOracle.

---

### 🎲 3. **Fuzz Testing**

**Goal:**
Automatically test your functions with **randomized input values**.
Foundry generates many inputs to find edge cases or unexpected behaviors.

**Example:**

* Check `transfer()` never reverts for any valid address and amount.
* Bound inputs within certain ranges using `bound()` helper.

---

### ♾️ 4. **Invariant Testing**

**Goal:**
Continuously test that **certain properties are always true**, no matter how many random actions occur.
Used for **system integrity** and **security verification**.

**Example:**

* `totalSupply == sum(userBalances)` must always hold.
* No one should withdraw more than deposited.


---

### 🌍 5. **Fork Testing**

**Goal:**
Run tests against **live blockchain state** (mainnet or testnet) by forking it.
Verifies compatibility with deployed contracts and real data.

**Example:**

* Test your DeFi strategy using real Uniswap pools.
* Simulate interactions with live Chainlink feeds.


---

### 🧰 6. **Mocks**

**Goal:**
Simulate or replace real external contracts (like oracles or tokens) to isolate test logic.
Mocks are not tests themselves, but support other test types.

**Example:**

* `MockERC20.sol`
* `MockOracle.sol`
