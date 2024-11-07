# ICP Token Wallet - Rust Implementation

This project demonstrates the creation of an Internet Computer Protocol (ICP) based token wallet using Rust. This wallet supports the basic functionalities of sending and receiving IRCRC2 tokens, displaying balances, and implementing basic security features. The code is written to be deployed on a local ICP test network.

## Table of Contents
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Smart Contract Design](#smart-contract-design)
- [Wallet Features](#wallet-features)
- [Unit Tests](#unit-tests)
- [Documentation](#documentation)
- [Security Considerations](#security-considerations)
- [Conclusion](#conclusion)

---

## Prerequisites

To run and deploy this wallet, you will need the following:

1. **Rust**: Ensure that Rust is installed. You can install it using [Rustup](https://rustup.rs/).
2. **DFX SDK**: The DFINITY SDK (`dfx`) is required to interact with the Internet Computer. Install it from [here](https://internetcomputer.org/docs/current/developer-docs/build/install-upgrade-remove).
3. **ICP Local Testnet**: Set up a local development environment for the ICP blockchain using the DFX SDK.
4. **Git**: For version control.

```bash
# Install Rust
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

# Install DFX SDK
sh -ci "$(curl -fsSL https://internetcomputer.org/install.sh)"
```

## Installation

### Step 1: Clone the Repository

```bash
git clone https://github.com/your-username/icp-token-wallet.git
cd icp-token-wallet
```

### Step 2: Start a Local ICP Network

In a new terminal, start the local ICP testnet:

```bash
dfx start --background
```

### Step 3: Deploy the Smart Contract

From the project directory, deploy the wallet smart contract:

```bash
dfx deploy
```

This will deploy the wallet canister to the local testnet, and you can interact with it using the `dfx` command-line tool.

## Smart Contract Design

The ICP token wallet is implemented in Rust using the [Candid](https://github.com/dfinity/candid) interface definition language. The core functionalities include sending tokens, receiving tokens, balance management, and basic wallet security.

### Contract Components

- **Token Structure**: A basic structure to represent the token, including owner and balance.
- **Send Function**: Allows users to send tokens to other addresses.
- **Receive Function**: Handles receiving tokens and updating the balance.
- **Balance Function**: Displays the current token balance of the wallet.

### Code Snippets

#### 1. Token Structure

```rust
#[derive(Clone, Debug, Default)]
pub struct Token {
    pub owner: Principal,
    pub balance: u64,
}
```

#### 2. Send Tokens

```rust
#[update]
async fn send_tokens(to: Principal, amount: u64) -> Result<(), String> {
    let caller = ic::caller();

    if let Some(sender_token) = TOKENS.get_mut(&caller) {
        if sender_token.balance < amount {
            return Err("Insufficient balance".to_string());
        }

        sender_token.balance -= amount;

        let recipient_token = TOKENS.entry(to).or_insert(Token {
            owner: to,
            balance: 0,
        });
        recipient_token.balance += amount;

        Ok(())
    } else {
        Err("Sender doesn't own any tokens".to_string())
    }
}
```

#### 3. Receive Tokens

```rust
#[update]
async fn receive_tokens(from: Principal, amount: u64) -> Result<(), String> {
    let caller = ic::caller();

    let recipient_token = TOKENS.entry(caller).or_insert(Token {
        owner: caller,
        balance: 0,
    });

    recipient_token.balance += amount;

    Ok(())
}
```

#### 4. Balance Display

```rust
#[query]
fn get_balance(owner: Principal) -> u64 {
    TOKENS.get(&owner).map_or(0, |t| t.balance)
}
```

## Wallet Features

### 1. **Send Tokens**

Users can send tokens to another wallet by specifying the recipient's principal ID and the amount of tokens to send.

- The `send_tokens` function checks if the sender has enough balance and then deducts the appropriate amount while adding it to the recipient's balance.

### 2. **Receive Tokens**

Tokens can be received by calling the `receive_tokens` function, which updates the wallet balance of the recipient.

### 3. **Balance Display**

Users can view their current token balance by calling the `get_balance` function with their principal ID.

## Unit Tests

Unit tests are written to ensure the correctness of the wallet's functionality, specifically the sending and receiving of tokens, as well as balance checks.

### Example Test

```rust
#[test]
fn test_send_tokens() {
    let sender = Principal::from_text("sender-principal-id").unwrap();
    let receiver = Principal::from_text("receiver-principal-id").unwrap();

    // Initialize sender with tokens
    TOKENS.insert(sender, Token { owner: sender, balance: 100 });

    // Send tokens from sender to receiver
    let result = send_tokens(receiver, 50);
    assert!(result.is_ok());

    // Check balances
    assert_eq!(get_balance(sender), 50);
    assert_eq!(get_balance(receiver), 50);
}
```

To run the tests:

```bash
cargo test
```

## Documentation

### How to Send Tokens

1. Deploy the contract.
2. Use the following command to send tokens:

```bash
dfx canister call <canister-id> send_tokens '(principal "<recipient-id>", <amount>)'
```

### How to Receive Tokens

Invoke the `receive_tokens` function using:

```bash
dfx canister call <canister-id> receive_tokens '(principal "<sender-id>", <amount>)'
```

### How to Check Balance

Check the balance with:

```bash
dfx canister call <canister-id> get_balance '(principal "<your-id>")'
```

## Security Considerations

1. **Access Control**: Only the owner of a token can transfer it. This is enforced by checking the principal ID of the caller.
2. **Balance Validation**: The smart contract ensures that a user cannot send more tokens than they own.
3. **Transaction Integrity**: The balance of both sender and receiver is updated atomically to avoid inconsistencies.

## Conclusion

This Rust-based token wallet demonstrates the basic principles of blockchain development on the Internet Computer Protocol (ICP) platform. The wallet allows users to send and receive IRCRC2 tokens, check balances, and ensures basic security measures are in place to protect transactions.

---

### Future Improvements

- **Token Approvals**: Implement a mechanism to allow third parties to spend tokens on behalf of the user.
- **Events**: Emit events for transactions to improve traceability.
- **UI Integration**: Build a front-end interface to interact with the wallet easily.
