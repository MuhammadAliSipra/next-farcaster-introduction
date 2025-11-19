# Wallet Connection in Farcaster Mini Apps - Beginner's Guide

A simple, step-by-step guide to connecting crypto wallets in your Farcaster mini app.

---

## 📚 Table of Contents
1. [What is Wallet Connection?](#what-is-wallet-connection)
2. [Why Do You Need It?](#why-do-you-need-it)
3. [Prerequisites](#prerequisites)
4. [Step-by-Step Implementation](#step-by-step-implementation)
5. [Code Examples](#code-examples)
6. [Common Issues & Solutions](#common-issues--solutions)
7. [Best Practices](#best-practices)

---

## 🤔 What is Wallet Connection?

A **wallet connection** allows your mini app to interact with a user's crypto wallet. Think of it like:
- **Logging in** - But with a crypto wallet instead of email/password
- **Payment method** - Like connecting your credit card to an app
- **Identity verification** - Proving you own a specific wallet address

**In Farcaster mini apps, wallet connection lets you:**
- Accept cryptocurrency payments (USDC, ETH, etc.)
- Verify user ownership of NFTs or tokens
- Interact with smart contracts
- Create blockchain-based features

---

## ❓ Why Do You Need It?

**You DON'T need wallet connection if you're building:**
- A simple poll or voting app
- A content viewer
- A game without payments
- An information display app

**You DO need wallet connection if you're building:**
- An app that accepts payments
- An NFT minting app
- A DeFi feature
- Anything requiring on-chain transactions

---

## ✅ Prerequisites

Before you start, make sure you have:

1. **Basic Next.js app** set up
2. **Farcaster MiniKit SDK** installed
   ```bash
   npm install @farcaster/frame-sdk
   ```
3. **TypeScript** configured (recommended)
4. **Understanding of React hooks** (useState, useEffect)

---

## 🛠️ Step-by-Step Implementation

### Step 1: Install Required Dependencies

```bash
# Install MiniKit SDK (if not already installed)
npm install @farcaster/frame-sdk

# Install ethers.js for blockchain interactions (optional but recommended)
npm install ethers
```

### Step 2: Set Up MiniKit Provider

The **MiniKit Provider** wraps your app and provides wallet functionality.

**Create a Providers component:**

```typescript
// src/components/Providers.tsx
'use client';

import { useEffect } from 'react';
import sdk from '@farcaster/frame-sdk';

export function Providers({ children }: { children: React.ReactNode }) {
  useEffect(() => {
    // Initialize the SDK
    const load = async () => {
      await sdk.actions.ready();
    };
    load();
  }, []);

  return <>{children}</>;
}
```

**Add it to your layout:**

```typescript
// src/app/layout.tsx
import { Providers } from '@/components/Providers';

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <body>
        <Providers>
          {children}
        </Providers>
      </body>
    </html>
  );
}
```

### Step 3: Create a Wallet Connect Button Component

This is the button users will click to connect their wallet.

**Create the component:**

```typescript
// src/components/ConnectWalletButton.tsx
'use client';

import { useState, useEffect } from 'react';
import sdk from '@farcaster/frame-sdk';

export function ConnectWalletButton() {
  const [isConnected, setIsConnected] = useState(false);
  const [walletAddress, setWalletAddress] = useState<string>('');
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState<string>('');

  // Connect wallet function
  const connectWallet = async () => {
    try {
      setIsLoading(true);
      setError('');

      // Request wallet connection from MiniKit
      const result = await sdk.wallet.ethProvider.request({
        method: 'eth_requestAccounts',
      });

      if (result && result.length > 0) {
        const address = result[0];
        setWalletAddress(address);
        setIsConnected(true);
        console.log('Connected wallet:', address);
      }
    } catch (err) {
      console.error('Error connecting wallet:', err);
      setError('Failed to connect wallet. Please try again.');
    } finally {
      setIsLoading(false);
    }
  };

  // Disconnect wallet function
  const disconnectWallet = () => {
    setWalletAddress('');
    setIsConnected(false);
    setError('');
  };

  // Check if wallet is already connected on mount
  useEffect(() => {
    const checkConnection = async () => {
      try {
        const accounts = await sdk.wallet.ethProvider.request({
          method: 'eth_accounts',
        });
        
        if (accounts && accounts.length > 0) {
          setWalletAddress(accounts[0]);
          setIsConnected(true);
        }
      } catch (err) {
        console.error('Error checking connection:', err);
      }
    };

    checkConnection();
  }, []);

  // Helper function to shorten wallet address for display
  const shortenAddress = (address: string) => {
    return `${address.slice(0, 6)}...${address.slice(-4)}`;
  };

  return (
    <div className="flex flex-col items-center gap-4">
      {!isConnected ? (
        <button
          onClick={connectWallet}
          disabled={isLoading}
          className="px-6 py-3 bg-blue-600 text-white rounded-lg font-semibold hover:bg-blue-700 disabled:bg-gray-400 disabled:cursor-not-allowed transition-colors"
        >
          {isLoading ? 'Connecting...' : 'Connect Wallet'}
        </button>
      ) : (
        <div className="flex flex-col items-center gap-2">
          <div className="px-4 py-2 bg-green-100 text-green-800 rounded-lg font-mono text-sm">
            ✅ Connected: {shortenAddress(walletAddress)}
          </div>
          <button
            onClick={disconnectWallet}
            className="px-4 py-2 text-sm text-red-600 hover:text-red-800 transition-colors"
          >
            Disconnect
          </button>
        </div>
      )}

      {error && (
        <div className="px-4 py-2 bg-red-100 text-red-800 rounded-lg text-sm">
          {error}
        </div>
      )}
    </div>
  );
}
```

### Step 4: Use the Component in Your App

```typescript
// src/app/page.tsx
import { ConnectWalletButton } from '@/components/ConnectWalletButton';

export default function Home() {
  return (
    <main className="flex min-h-screen flex-col items-center justify-center p-8">
      <h1 className="text-4xl font-bold mb-8">My Farcaster Mini App</h1>
      <ConnectWalletButton />
    </main>
  );
}
```

### Step 5: Create a Custom Hook (Advanced - Optional)

For better code organization, create a custom hook to manage wallet state:

```typescript
// src/hooks/useWallet.ts
'use client';

import { useState, useEffect } from 'react';
import sdk from '@farcaster/frame-sdk';

export function useWallet() {
  const [isConnected, setIsConnected] = useState(false);
  const [walletAddress, setWalletAddress] = useState<string>('');
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState<string>('');

  // Connect wallet
  const connect = async () => {
    try {
      setIsLoading(true);
      setError('');

      const result = await sdk.wallet.ethProvider.request({
        method: 'eth_requestAccounts',
      });

      if (result && result.length > 0) {
        setWalletAddress(result[0]);
        setIsConnected(true);
      }
    } catch (err) {
      setError('Failed to connect wallet');
      console.error(err);
    } finally {
      setIsLoading(false);
    }
  };

  // Disconnect wallet
  const disconnect = () => {
    setWalletAddress('');
    setIsConnected(false);
    setError('');
  };

  // Check existing connection
  useEffect(() => {
    const checkConnection = async () => {
      try {
        const accounts = await sdk.wallet.ethProvider.request({
          method: 'eth_accounts',
        });
        
        if (accounts && accounts.length > 0) {
          setWalletAddress(accounts[0]);
          setIsConnected(true);
        }
      } catch (err) {
        console.error('Error checking connection:', err);
      }
    };

    checkConnection();
  }, []);

  return {
    isConnected,
    walletAddress,
    isLoading,
    error,
    connect,
    disconnect,
  };
}
```

**Using the custom hook:**

```typescript
// src/components/ConnectWalletButton.tsx (Simplified version)
'use client';

import { useWallet } from '@/hooks/useWallet';

export function ConnectWalletButton() {
  const { isConnected, walletAddress, isLoading, error, connect, disconnect } = useWallet();

  const shortenAddress = (address: string) => {
    return `${address.slice(0, 6)}...${address.slice(-4)}`;
  };

  return (
    <div className="flex flex-col items-center gap-4">
      {!isConnected ? (
        <button
          onClick={connect}
          disabled={isLoading}
          className="px-6 py-3 bg-blue-600 text-white rounded-lg font-semibold hover:bg-blue-700 disabled:bg-gray-400"
        >
          {isLoading ? 'Connecting...' : 'Connect Wallet'}
        </button>
      ) : (
        <div className="flex flex-col items-center gap-2">
          <div className="px-4 py-2 bg-green-100 text-green-800 rounded-lg font-mono text-sm">
            ✅ {shortenAddress(walletAddress)}
          </div>
          <button onClick={disconnect} className="text-sm text-red-600 hover:text-red-800">
            Disconnect
          </button>
        </div>
      )}
      {error && <div className="text-red-600 text-sm">{error}</div>}
    </div>
  );
}
```

---

## 💰 Making Transactions (After Connection)

Once the wallet is connected, you can request transactions:

```typescript
// Example: Send ETH payment
async function sendPayment() {
  try {
    const transactionHash = await sdk.wallet.ethProvider.request({
      method: 'eth_sendTransaction',
      params: [{
        to: '0xRecipientAddress',
        value: '0x9184e72a000', // Amount in hex (0.01 ETH)
        from: walletAddress, // User's connected wallet
      }],
    });
    
    console.log('Transaction sent:', transactionHash);
    return transactionHash;
  } catch (error) {
    console.error('Transaction failed:', error);
    throw error;
  }
}
```

---

## 🔧 Common Issues & Solutions

### Issue 1: "SDK is not initialized"
**Solution:** Make sure you call `sdk.actions.ready()` before using wallet features.

```typescript
useEffect(() => {
  const init = async () => {
    await sdk.actions.ready();
    // Now you can use wallet features
  };
  init();
}, []);
```

### Issue 2: "eth_requestAccounts returned empty array"
**Solution:** The user might not have a wallet set up in Warpcast. Guide them to:
1. Open Warpcast settings
2. Go to "Wallets"
3. Connect or add a wallet

### Issue 3: "Cannot read properties of undefined"
**Solution:** Always check if SDK is available before using it:

```typescript
if (sdk && sdk.wallet && sdk.wallet.ethProvider) {
  // Safe to use
  await sdk.wallet.ethProvider.request({...});
}
```

### Issue 4: Transaction fails silently
**Solution:** Add proper error handling and user feedback:

```typescript
try {
  const result = await sdk.wallet.ethProvider.request({...});
  alert('Success!');
} catch (error) {
  if (error.code === 4001) {
    alert('Transaction rejected by user');
  } else {
    alert('Transaction failed: ' + error.message);
  }
}
```

### Issue 5: Wallet disconnects on page refresh
**Solution:** Check for existing connection on component mount (already shown in Step 3).

---

## ✨ Best Practices

### 1. **Always Show Connection Status**
Users should always know if they're connected:

```typescript
{isConnected ? (
  <div className="text-green-600">✅ Wallet Connected</div>
) : (
  <div className="text-gray-600">❌ Not Connected</div>
)}
```

### 2. **Handle Loading States**
Show loading indicators during connection:

```typescript
<button disabled={isLoading}>
  {isLoading ? (
    <span>Connecting... ⏳</span>
  ) : (
    <span>Connect Wallet</span>
  )}
</button>
```

### 3. **Provide Clear Error Messages**
Don't just show technical errors:

```typescript
// Bad
setError(error.message);

// Good
setError('Unable to connect. Please make sure you have a wallet set up in Warpcast.');
```

### 4. **Shorten Wallet Addresses**
Full addresses are hard to read on mobile:

```typescript
// Display: 0x1234...5678 instead of 0x1234567890abcdef1234567890abcdef12345678
const shortenAddress = (addr: string) => `${addr.slice(0, 6)}...${addr.slice(-4)}`;
```

### 5. **Persist Connection State**
Check for existing connection when component mounts (already shown in examples).

### 6. **Test on Real Devices**
Always test wallet connection on actual mobile devices with Warpcast app.

### 7. **Handle Network Switching**
Farcaster mini apps primarily use Base network:

```typescript
const BASE_CHAIN_ID = '0x2105'; // Base mainnet

async function ensureCorrectNetwork() {
  try {
    await sdk.wallet.ethProvider.request({
      method: 'wallet_switchEthereumChain',
      params: [{ chainId: BASE_CHAIN_ID }],
    });
  } catch (error) {
    console.error('Failed to switch network:', error);
  }
}
```

### 8. **Add TypeScript Types**
Make your code safer with proper types:

```typescript
interface WalletState {
  isConnected: boolean;
  walletAddress: string;
  chainId?: string;
  balance?: string;
}

const [wallet, setWallet] = useState<WalletState>({
  isConnected: false,
  walletAddress: '',
});
```

---

## 📱 Testing Your Wallet Connection

### On Desktop (Development)
1. Run your app: `npm run dev`
2. Open browser console
3. Check for any SDK errors
4. Note: Full wallet features only work in Warpcast mobile app

### On Mobile (Real Testing)
1. Deploy your app to Vercel
2. Open Warpcast mobile app
3. Create a cast with your app URL
4. Tap to open as mini app
5. Test wallet connection
6. Try disconnecting and reconnecting
7. Test with and without wallet set up

---

## 🎯 Quick Reference

### Connect Wallet
```typescript
const accounts = await sdk.wallet.ethProvider.request({
  method: 'eth_requestAccounts',
});
```

### Check Current Connection
```typescript
const accounts = await sdk.wallet.ethProvider.request({
  method: 'eth_accounts',
});
```

### Get Wallet Balance
```typescript
const balance = await sdk.wallet.ethProvider.request({
  method: 'eth_getBalance',
  params: [walletAddress, 'latest'],
});
```

### Send Transaction
```typescript
const txHash = await sdk.wallet.ethProvider.request({
  method: 'eth_sendTransaction',
  params: [{
    to: recipientAddress,
    value: amountInHex,
    from: walletAddress,
  }],
});
```

---

## 📚 Next Steps

After implementing wallet connection, you can:

1. **Add Payment Features**
   - Accept USDC payments
   - Implement credit purchase system
   - See: `CREDIT_SYSTEM.md`

2. **Interact with Smart Contracts**
   - Mint NFTs
   - Call contract functions
   - Read blockchain data

3. **Add Token Features**
   - Check token balances
   - Verify NFT ownership
   - Gate content by token holdings

4. **Implement Transactions**
   - Send cryptocurrency
   - Swap tokens
   - DeFi interactions

---

## 🆘 Need Help?

**Resources:**
- [Farcaster MiniKit Docs](https://docs.farcaster.xyz/developers/minikit)
- [MiniKit Wallet API](https://docs.farcaster.xyz/developers/minikit/wallet)
- Farcaster Developer Communities
- Stack Overflow (tag: farcaster)

**Common Questions:**
- Q: Do users need to connect wallet for every visit?
  - A: No, if they stay in Warpcast, the connection persists.

- Q: Which networks are supported?
  - A: Primarily Base network (Ethereum L2), but also Ethereum mainnet.

- Q: Can I use other wallet libraries like wagmi or RainbowKit?
  - A: For Farcaster mini apps, it's recommended to use MiniKit SDK.

- Q: Is wallet connection secure?
  - A: Yes, MiniKit uses secure methods and users approve all transactions.

---

## ✅ Checklist

Before going to production, ensure:

- [ ] Wallet connection works in Warpcast mobile app
- [ ] Loading states are shown
- [ ] Error messages are user-friendly
- [ ] Connection persists between pages
- [ ] Disconnect function works
- [ ] Wallet address is displayed correctly
- [ ] No console errors
- [ ] Tested with and without wallet setup
- [ ] Transaction flows work (if applicable)
- [ ] Proper error handling for failed transactions

---

**Congratulations! 🎉** You now know how to implement wallet connection in Farcaster mini apps.

Start simple, test thoroughly, and iterate based on user feedback. Good luck! 🚀

