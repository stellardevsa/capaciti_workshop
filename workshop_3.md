# Stellar Workshop 3

Welcome to this hands-on workshop where you'll build your first decentralized marketplace on the Stellar blockchain! By the end of this tutorial, you'll have a working web application that can connect to Stellar wallets and process XLM payments.

## What You'll Build

A simple marketplace where users can:
- Connect their Stellar wallet (xBull or Freighter)
- Browse products
- Make payments in XLM (Stellar Lumens)
- See transaction confirmations

## Prerequisites

- Basic understanding of HTML/CSS
- Node.js installed on your computer
- A code editor (VS Code recommended)
- A Stellar wallet (we'll help you set one up)

## Step 1: Project Setup

### Create Your Project

Open your terminal and run these commands:

```bash
# Create a new React project
npm create vite@latest marketplace --template react

# Navigate to the project folder
cd marketplace

# Install required packages
npm install --save @stellar/stellar-sdk
npm install @creit.tech/stellar-wallets-kit

# Start the development server
npm run dev
```

Your project should now be running at `http://localhost:5173`

### Install a Stellar Wallet

Before we code, you'll need a wallet to test with:

1. **Freighter**: Install the browser extension from [freighter.app](https://freighter.app)
2. **xBull**: Install from [xbull.app](https://xbull.app)

Create a new wallet and **make sure to switch to TESTNET** in the wallet settings.

## Step 2: Understanding the Project Structure

Your project will have this structure:

```
marketplace/
├── src/
│   ├── components/
│   │   ├── WalletConnector.jsx
│   │   ├── Product.jsx
│   │   └── ProductList.jsx
│   ├── App.jsx
│   ├── App.css
│   └── main.jsx
└── package.json
```

## Step 3: Configure Vite (vite.config.js)

Replace the contents of `vite.config.js` with:

```javascript
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()],
  define: {
    global: 'window',
  },
})
```

**Why this matters**: This configuration tells Vite to treat `global` as `window`, which is needed for the Stellar SDK to work in the browser.

## Step 4: Create the Wallet Connector Component

Create `src/components/WalletConnector.jsx`:

```javascript
import { useState } from 'react';

function WalletConnector({ onConnect }) {
  const wallets = [
    { id: 'xbull', name: 'xBull' },
    { id: 'freighter', name: 'Freighter' },
  ];

  const handleSelectWallet = (walletId) => {
    onConnect(walletId);
  };

  return (
    <div className="wallet-connector">
      <h2>Connect Your Wallet</h2>
      <div>
        {wallets.map((wallet) => (
          <button 
            key={wallet.id} 
            onClick={() => handleSelectWallet(wallet.id)}
          >
            Connect with {wallet.name}
          </button>
        ))}
      </div>
    </div>
  );
}

export default WalletConnector;
```

**What this does**: Creates buttons for different wallet types that users can click to connect.

## Step 5: Create the Product Component

Create `src/components/Product.jsx`:

```javascript
import { useState } from 'react';
import { TransactionBuilder, Operation, Asset, Networks } from '@stellar/stellar-sdk';

function Product({ product, publicKey, kit, server, setStatus }) {
  const [localStatus, setLocalStatus] = useState('');

  const handleBuy = async () => {
    setLocalStatus('Processing...');
    setStatus('Processing payment...');

    try {
      // Step 1: Load buyer's account from Stellar network
      const account = await server.loadAccount(publicKey);

      // Step 2: Build the payment transaction
      const transaction = new TransactionBuilder(account, {
        fee: await server.fetchBaseFee(),
        networkPassphrase: Networks.TESTNET,
      })
      .addOperation(
        Operation.payment({
          destination: product.seller,
          asset: Asset.native(), // XLM
          amount: product.price,
        })
      )
      .setTimeout(30)
      .build();

      // Step 3: Sign transaction with user's wallet
      const { signedTxXdr } = await kit.signTransaction(transaction.toXDR(), {
        address: publicKey,
        networkPassphrase: Networks.TESTNET,
      });

      // Step 4: Submit transaction to Stellar network
      const signedTransaction = TransactionBuilder.fromXDR(signedTxXdr, Networks.TESTNET);
      await server.submitTransaction(signedTransaction);

      setLocalStatus('Purchase successful!');
      setStatus(`Successfully bought ${product.name}!`);
    } catch (error) {
      console.error('Purchase failed:', error);
      setLocalStatus('Purchase failed.');
      setStatus('Payment failed. Check console for details.');
    }
  };

  return (
    <div className="product">
      <h3>{product.name}</h3>
      <p>Price: {product.price} XLM</p>
      <button onClick={handleBuy}>Buy</button>
      <p>{localStatus}</p>
    </div>
  );
}

export default Product;
```

**What this does**: 
- Displays product information
- Handles the complete payment process when "Buy" is clicked
- Shows status updates during the transaction

## Step 6: Create the Product List Component

Create `src/components/ProductList.jsx`:

```javascript
import Product from './Product';

const products = [
  { 
    id: 1, 
    name: 'Stellar T-Shirt', 
    price: '5', 
    seller: 'GASAFXGOF7BY6NDVESCDERFBUBTF4SHCJWO7GWWM3YPOEDUAIUXNF3ZD' 
  },
  { 
    id: 2, 
    name: 'XLM Mug', 
    price: '10', 
    seller: 'GASAFXGOF7BY6NDVESCDERFBUBTF4SHCJWO7GWWM3YPOEDUAIUXNF3ZD' 
  },
];

function ProductList({ publicKey, kit, server, setStatus }) {
  return (
    <div className="product-list">
      <h2>Available Products</h2>
      {products.map((product) => (
        <Product
          key={product.id}
          product={product}
          publicKey={publicKey}
          kit={kit}
          server={server}
          setStatus={setStatus}
        />
      ))}
    </div>
  );
}

export default ProductList;
```

**What this does**: Lists all available products and renders a Product component for each one.

## Step 7: Create the Main App Component

Replace `src/App.jsx` with:

```javascript
import { useState } from 'react';
import { 
  StellarWalletsKit, 
  WalletNetwork, 
  xBullModule, 
  FreighterModule, 
  XBULL_ID 
} from '@creit.tech/stellar-wallets-kit';
import { Horizon, Networks } from '@stellar/stellar-sdk';
import WalletConnector from './components/WalletConnector';
import ProductList from './components/ProductList';
import './App.css';

// Initialize the wallet kit (this connects to different wallet types)
const kit = new StellarWalletsKit({
  network: WalletNetwork.TESTNET,
  selectedWalletId: XBULL_ID,
  modules: [
    new xBullModule(),
    new FreighterModule()
  ],
});

// Initialize connection to Stellar testnet
const server = new Horizon.Server('https://horizon-testnet.stellar.org');

function App() {
  const [publicKey, setPublicKey] = useState(null);
  const [status, setStatus] = useState('');

  const handleConnect = async (walletId) => {
    try {
      await kit.setWallet(walletId);
      const { address } = await kit.getAddress();
      setPublicKey(address);
      setStatus(`Connected with public key: ${address.slice(0, 6)}...`);
    } catch (error) {
      console.error('Connection failed:', error);
      setStatus('Failed to connect wallet.');
    }
  };

  return (
    <div className="app">
      <h1>Simple Stellar Marketplace (Testnet)</h1>
      <WalletConnector onConnect={handleConnect} />
      <p>{status}</p>
      
      {publicKey && (
        <ProductList 
          publicKey={publicKey}
          kit={kit}
          server={server}
          setStatus={setStatus}
        />
      )}
      
      <p className="note">
        Need test XLM? Use{' '}
        <a 
          href="https://friendbot.stellar.org" 
          target="_blank" 
          rel="noopener noreferrer"
        >
          Friendbot
        </a>.
      </p>
    </div>
  );
}

export default App;
```

**What this does**: 
- Sets up the main application
- Manages wallet connection state
- Only shows products after wallet is connected
- Provides link to get test XLM

## Step 8: Add Some Basic Styling

Replace `src/App.css` with:

```css
.app {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
  font-family: Arial, sans-serif;
}

.wallet-connector {
  margin-bottom: 20px;
  padding: 20px;
  border: 1px solid #ddd;
  border-radius: 8px;
}

.wallet-connector button {
  margin: 5px;
  padding: 10px 20px;
  background-color: #007bff;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.wallet-connector button:hover {
  background-color: #0056b3;
}

.product-list {
  margin-top: 20px;
}

.product {
  border: 1px solid #ddd;
  border-radius: 8px;
  padding: 15px;
  margin: 10px 0;
  background-color: #f9f9f9;
}

.product button {
  background-color: #28a745;
  color: white;
  border: none;
  padding: 8px 16px;
  border-radius: 4px;
  cursor: pointer;
}

.product button:hover {
  background-color: #218838;
}

.note {
  margin-top: 20px;
  padding: 10px;
  background-color: #fff3cd;
  border-radius: 4px;
  font-size: 14px;
}
```

## Step 9: Test Your Application

1. **Start the development server**:
   ```bash
   npm run dev
   ```

2. **Get test XLM**:
   - Copy your wallet's public key
   - Visit [https://friendbot.stellar.org](https://friendbot.stellar.org)
   - Paste your public key and click "Fund with Friendbot"

3. **Test the marketplace**:
   - Click "Connect with Freighter" (or xBull)
   - Approve the connection in your wallet
   - Try buying a product
   - Confirm the transaction in your wallet

## Understanding Key Concepts

### What is Stellar?
Stellar is a blockchain network designed for fast, low-cost cross-border payments. XLM (Lumens) is its native cryptocurrency.

### What are we doing?
- **Connecting wallets**: Using the Stellar Wallets Kit to connect to user wallets
- **Building transactions**: Creating payment operations using the Stellar SDK
- **Signing transactions**: Getting user approval through their wallet
- **Submitting transactions**: Broadcasting the signed transaction to the Stellar network

### Key Components Explained

1. **TransactionBuilder**: Creates Stellar transactions
2. **Operation.payment()**: Defines a payment from one account to another
3. **kit.signTransaction()**: Asks the user's wallet to sign the transaction
4. **server.submitTransaction()**: Sends the transaction to the Stellar network

## Common Issues and Solutions

**Wallet won't connect**: Make sure you're on testnet in your wallet settings

**Transaction fails**: Check if you have enough XLM balance (get more from Friendbot)

**App won't start**: Make sure all dependencies are installed (`npm install`)

## Next Steps

Now that you have a working marketplace, try these enhancements:

1. **Add more products** with different prices
2. **Show transaction history** using Stellar's transaction API
3. **Add custom tokens** instead of just XLM
4. **Implement escrow** for secure transactions
5. **Add user profiles** and seller ratings

## Congratulations!

You've successfully built your first decentralized marketplace on Stellar! You now understand:
- How to connect Stellar wallets to web applications
- How to build and sign transactions
- How to interact with the Stellar blockchain
- The basics of decentralized payment processing

This foundation will help you build more complex Stellar applications in the future.

## Hackathon Project Ideas organized by difficulty and real-world impact:

**Beginner Apps (Modified Marketplace)**

Digital Art Marketplace
Course/Tutorial Platform
Freelancer Services Hub
Event Ticketing System
Digital Collectibles Exchange

**Intermediate Apps**

- Carbon Credit Marketplace 
- Remittance Platform 
- Supply Chain Transparency
- Micro-Insurance Platform 
- Community Solar Marketplace
- Local Currency Exchange
- Subscription Management Platform
- Crowdfunding Platform
- Peer-to-Peer Lending
- Skills Verification System

**Advanced Apps (Complex Problems)**

- Decentralized Clinical Trials 
- Refugee Aid Distribution 
- Intellectual Property Exchange 
- Disaster Response Coordination 
- Democratic Voting Platform 
- Medical Records System
- Land Title Registry
- Identity Verification Platform
- Prediction Markets
- Decentralized Autonomous Organization (DAO)




