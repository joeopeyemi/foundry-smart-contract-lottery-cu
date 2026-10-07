# 🎰 Smart Contract Lottery — Foundry & Chainlink VRF

A decentralized raffle (lottery) smart contract built with **Solidity, Foundry, and Chainlink VRF v2.5**.

The contract allows users to enter a raffle by paying an entrance fee. After a configured time interval, Chainlink Automation can trigger the raffle to select a random winner using Chainlink VRF. The winner receives the entire raffle balance, and the raffle automatically resets for the next round.

## 🚀 Features

* Users can enter the raffle by paying the required entrance fee.
* Prevents users from entering when the raffle is calculating a winner.
* Uses **Chainlink VRF v2.5** for verifiable randomness.
* Uses automated upkeep logic to determine when a winner should be selected.
* Automatically selects a random winner.
* Transfers the entire raffle balance to the winner.
* Resets the raffle after a winner is selected.
* Includes local testing using Chainlink VRF v2.5 mocks.
* Supports deployment to a local Anvil chain and Ethereum Sepolia.
* Includes deployment and interaction scripts.
* Includes a comprehensive Foundry test suite.

---

## 🛠️ Tech Stack

* **Solidity:** `0.8.19`
* **Foundry**

  * Forge
  * Anvil
  * Cast
* **Chainlink VRF v2.5**
* **Chainlink Automation**
* **Ethereum Sepolia**
* **Git & GitHub**

---

## 📁 Project Structure

```text
foundry-smart-contract-lottery-cu/
│
├── src/
│   └── Raffle.sol
│
├── script/
│   ├── DeployRaffle.s.sol
│   ├── HelperConfig.s.sol
│   └── interactions.s.sol
│
├── test/
│   ├── RaffleTest.t.sol
│   └── mocks/
│       └── LinkToken.sol
│
├── lib/
│   ├── chainlink-brownie-contracts/
│   ├── forge-std/
│   ├── foundry-devops/
│   └── solmate/
│
├── Makefile
├── foundry.toml
└── README.md
```

---

## ⚙️ How the Raffle Works

### 1. Entering the Raffle

A user calls:

```solidity
enterRaffle()
```

and sends at least the required entrance fee.

The player's address is added to the list of participants and a `RaffleEntered` event is emitted.

If the user does not send enough ETH, the transaction reverts with:

```text
Raffle__SendMoreToEnterRaffle
```

---

### 2. Checking Whether Upkeep Is Needed

The `checkUpkeep()` function checks four conditions:

1. The required time interval has passed.
2. The raffle is currently open.
3. The contract has an ETH balance.
4. There is at least one player.

If all four conditions are true, upkeep is required.

```solidity
upkeepNeeded =
    timeHasPassed &&
    isOpen &&
    hasBalance &&
    hasPlayers;
```

---

### 3. Performing Upkeep

When `performUpkeep()` is called, the raffle first verifies that upkeep is actually needed.

The raffle state is then changed from:

```text
OPEN
```

to:

```text
CALCULATING
```

A Chainlink VRF request is created to obtain a random number.

This prevents additional players from entering while a winner is being calculated.

---

### 4. Selecting the Winner

Once Chainlink VRF fulfills the request, `fulfillRandomWords()` is called.

The random number is used to select a player:

```solidity
uint256 indexOfWinner = randomWords[0] % s_players.length;
```

The selected player becomes the recent winner.

---

### 5. Paying the Winner

The entire ETH balance of the raffle contract is transferred to the winner.

After payment:

* The player list is cleared.
* The raffle state returns to `OPEN`.
* The timestamp is updated.
* A `WinnerPicked` event is emitted.

The raffle is then ready for the next round.

---

# 🧪 Testing

The project contains tests covering the major raffle behaviors.

Run the complete test suite with:

```bash
forge test
```

The tests cover:

### Raffle Initialization

* Raffle starts in the `OPEN` state.

### Entering the Raffle

* Reverts when insufficient ETH is sent.
* Correctly records players.
* Emits the `RaffleEntered` event.

### Raffle State

* Players cannot enter while the raffle is calculating.

### Chainlink Automation / Upkeep

* `checkUpkeep()` returns false when there is no balance.
* `checkUpkeep()` returns false when the raffle is not open.
* `checkUpkeep()` returns false when the required interval has not passed.
* `checkUpkeep()` returns true when all required conditions are satisfied.
* `performUpkeep()` reverts when upkeep is not needed.
* `performUpkeep()` changes the raffle state and emits a VRF request.

### Random Winner Selection

* VRF fulfillment cannot occur without a valid request.
* A winner is selected from the participating players.
* The raffle resets after the winner is selected.
* The winner receives the raffle prize.
* The timestamp is updated for the next raffle round.

---

# 🔧 Installation

Clone the repository:

```bash
git clone <YOUR_GITHUB_REPOSITORY_URL>
cd foundry-smart-contract-lottery-cu
```

Install the required dependencies:

```bash
make install
```

Alternatively:

```bash
forge install cyfrin/foundry-devops@0.2.2
forge install smartcontractkit/chainlink-brownie-contracts@1.1.1
forge install foundry-rs/forge-std@v1.8.2
forge install transmissions11/solmate@v6
```

Build the project:

```bash
make build
```

or:

```bash
forge build
```

---

# 🧪 Run Tests

Run the test suite with:

```bash
make test
```

or:

```bash
forge test
```

For detailed test output:

```bash
forge test -vvvv
```

---

# 🖥️ Local Development

The project can be tested locally using **Anvil**.

Start a local Anvil blockchain:

```bash
anvil
```

The `HelperConfig` contract automatically creates the required local:

* VRF Coordinator mock
* LINK token mock
* Network configuration

when running on chain ID:

```text
31337
```

This allows the raffle to be tested without interacting with the real Chainlink network.

---

# 🌐 Sepolia Deployment

The project also supports deployment to the Ethereum Sepolia testnet.

Create a `.env` file containing your required environment variables:

```text
SEPOLIA_RPC_URL=your_rpc_url
ETHERSCAN_API_KEY=your_etherscan_api_key
```

> ⚠️ Never commit your `.env` file, private keys, API keys, or other sensitive credentials to GitHub.

Deploy using:

```bash
make deploy-sepolia
```

The deployment script:

1. Loads the network configuration.
2. Creates a Chainlink VRF subscription if required.
3. Funds the subscription.
4. Deploys the `Raffle` contract.
5. Adds the deployed raffle as a VRF consumer.
6. Broadcasts the deployment.
7. Verifies the contract on Etherscan.

---

# 🔗 Chainlink Integration

This project uses **Chainlink VRF v2.5** to provide verifiable randomness.

The raffle relies on a Chainlink VRF Coordinator to request and return random numbers.

The project also includes upkeep logic through:

```solidity
checkUpkeep()
```

and:

```solidity
performUpkeep()
```

which allows the raffle to determine when a new winner should be selected.

---

# 📜 Main Contract

The main contract is:

```text
src/Raffle.sol
```

The contract is responsible for:

* Managing raffle entries
* Tracking players
* Managing raffle state
* Requesting randomness
* Selecting the winner
* Transferring the prize
* Resetting the raffle

---

# 👨🏾‍💻 Author

**Dandelion**

Built as a practical smart contract development project using **Solidity, Foundry, Chainlink VRF, and Ethereum Sepolia**.
