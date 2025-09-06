# Stellar Workshop 2

---

# Session 2: Introduction to Stellar Blockchain and Testnet Setup

### Objective

By the end of this workshop, you'll understand:

* Stellar Consensus Protocol (on the slides)
* Stellar Stack (on the slides)
* What Stellar is and what it’s used for (on the slides)
* How to set up your JavaScript/React development environment
* How to send real XLM on the Testnet using code

---

## Part 1: Theory 

### What is Stellar?

Stellar is a **Layer-1 blockchain** designed to move money quickly and cheaply. It uses the **Stellar Consensus Protocol (SCP)** — a unique algorithm that enables decentralized agreement without mining or proof-of-work.

---

### What is XLM?

XLM (Lumens) is Stellar’s native cryptocurrency. It's used for:

* Paying transaction fees (super cheap: \~0.00001 XLM per operation)
* Creating and activating new accounts (1 XLM minimum balance)
* Acting as a bridge currency for asset conversions

---

### Key Features

*  **Fast transactions**: Usually settle in \~5 seconds
*  **Low fees**: Cost is negligible
*  **Global-ready**: Great for remittances, tokenization, lending, and even blockchain-based voting

---

### Tools We’ll Use

* `@stellar/stellar-sdk`: JavaScript SDK for working with Stellar
* **Friendbot**: Testnet faucet to get free XLM
* **Horizon API**: The gateway to Stellar blockchain data
* **Stellar Testnet**: A sandbox version of Stellar to test your code safely

---

## Part 2: Hands-On Setup

---

### Step 1: Create the Project

In your terminal, run:

```bash
npm create vite@latest payment -- --template react
cd payment
npm install --save @stellar/stellar-sdk
```

This creates a new React app with Vite and installs Stellar's SDK so we can talk to the blockchain from JavaScript.

---

### Step 2: Install Freighter Wallet and Fund Your Wallet Address on the TestNet

* Go to: [Freighter](https://www.freighter.app/), click **Add to Browser** and install the Freighter Chrome extension on your browser. Don't lose your seed phrase :)
* Go to: [Fund Account](https://lab.stellar.org/account/fund?$=network$id=testnet&label=Testnet&horizonUrl=https:////horizon-testnet.stellar.org&rpcUrl=https:////soroban-testnet.stellar.org&passphrase=Test%20SDF%20Network%20/;%20September%202015;;)
* Paste your **public key**
* Click **Submit**
* You’ll get 10,000 Testnet XLM instantly!

---

### Step 3: Create the Payment Script

In your project, create a file at:
-> `src/utils/stellar.js`

Paste this code:

```javascript
import * as StellarSdk from '@stellar/stellar-sdk';

export async function sendPayment(secretKey, destinationId, amount) {
  const sourceKeypair = StellarSdk.Keypair.fromSecret(secretKey); // Load your secret key
  const server = new StellarSdk.Horizon.Server('https://horizon-testnet.stellar.org'); // Connect to Testnet

  const account = await server.loadAccount(sourceKeypair.publicKey()); // Get your account info
  const transaction = new StellarSdk.TransactionBuilder(account, {
    fee: StellarSdk.BASE_FEE, // Default fee for transactions
    networkPassphrase: StellarSdk.Networks.TESTNET, // Use Testnet, not Mainnet
  })
    .addOperation(StellarSdk.Operation.payment({
      destination: destinationId, // Receiver’s public key
      asset: StellarSdk.Asset.native(), // Native = XLM
      amount, // Amount to send as a string
    }))
    .setTimeout(30) // Set timeout in seconds
    .build(); // Build the transaction

  transaction.sign(sourceKeypair); // Sign using your secret key
  const result = await server.submitTransaction(transaction); // Submit to the blockchain
  return result; // Return the result
}
```

---

### Explanation of the Code

1. **`Keypair.fromSecret(secretKey)`**
   ➤ Loads your Stellar account using your secret key. You **must keep this secret key private!**

2. **`Horizon.Server(...)`**
   ➤ Connects to the Stellar Testnet using Horizon (the API layer for Stellar).

3. **`server.loadAccount(...)`**
   ➤ Fetches your account info from the blockchain so you can build a valid transaction.

4. **`TransactionBuilder`**
   ➤ Used to construct the transaction step-by-step. You must:

   * Define the source account
   * Add operations (like `payment`)
   * Set a timeout
   * Specify the network (Testnet here)

5. **`Operation.payment({...})`**
   ➤ This is the actual action — sending XLM from one account to another.

6. **`transaction.sign(...)`**
   ➤ You sign the transaction to authorize it.

7. **`server.submitTransaction(...)`**
   ➤ Sends the signed transaction to the Stellar blockchain.

---

### Step 4: Test the Payment Script

In the project root, create a file named `test-payment.js`:

```javascript
import { sendPayment } from './src/utils/stellar.js';

async function test() {
  try {
    const result = await sendPayment(
      'YOUR_SECRET_KEY',            // Replace this with the sender’s secret key
      'DESTINATION_PUBLIC_KEY',     // Replace with the receiver’s public key
      '10'                           // Send 10 XLM
    );
    console.log('Payment successful:', result);
  } catch (error) {
    console.error('Error:', error.message);
  }
}

test();
```

To run it:

```bash
node test-payment.js
```

If it works, you’ll see a success message and transaction details printed in your terminal.

---

## Exercise for Students

1. Generate two Testnet accounts (use `StellarSdk.Keypair.random()`).
2. Fund both accounts with Friendbot.
3. Use the `sendPayment()` function to send **5 XLM** from one account to the other.
4. Confirm the transaction using the returned result or check it on [https://testnet.stellar.expert](https://testnet.stellar.expert).

---

## Summary

Today, you learned:

* The fundamentals of Stellar and its use cases
* How to set up a React app using Vite
* How to fund and interact with the Stellar Testnet
* How to send XLM using the JavaScript SDK

You’ve now made your first blockchain transaction from code!


