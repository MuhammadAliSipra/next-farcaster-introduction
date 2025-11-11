# Making Your Next.js App Compatible with Farcaster Mini Apps

## What are Farcaster Mini Apps?

Farcaster Mini Apps are web applications that run inside the Farcaster social network. Think of them as lightweight apps that users can access directly from their Farcaster feed—no app store needed! Users are already logged in, and they can share your app with just one click.

**Why use Mini Apps?**
- 🚀 **No app store approval** - Ship instantly
- 👥 **Built-in social features** - Users are pre-authenticated
- 💰 **Easy payments** - Integrated crypto wallet
- 🔔 **Push notifications** - Re-engage users
- 📈 **Viral growth** - Easy sharing in social feeds

---

## Prerequisites

- Basic knowledge of Next.js
- A Next.js application (can be existing or new)
- Your app needs to be deployed (Vercel, Railway, etc.)

---

## Step-by-Step Setup Guide

### 1. Create the Farcaster Manifest File

Every Farcaster Mini App needs a manifest file at `.well-known/farcaster.json`. This tells Farcaster about your app.

**Create this file structure in your Next.js project:**
```
public/
  .well-known/
    farcaster.json
```

**Add this content to `farcaster.json`:**

```json
{
  "accountAssociation": {
    "header": "your-app-signature-here",
    "payload": "your-app-payload-here"
  },
  "frame": {
    "version": "1",
    "name": "Your App Name",
    "iconUrl": "https://yourdomain.com/icon.png",
    "homeUrl": "https://yourdomain.com",
    "splashImageUrl": "https://yourdomain.com/splash.png",
    "splashBackgroundColor": "#ffffff",
    "webhookUrl": "https://yourdomain.com/api/webhook"
  }
}
```

**Important fields to customize:**
- `name` - Your app's display name (max 20 characters)
- `iconUrl` - Square app icon, at least 200x200px (PNG or JPG)
- `homeUrl` - The URL where your app lives
- `splashImageUrl` - Loading screen image (1500x1500px recommended)
- `splashBackgroundColor` - Hex color for the loading screen
- `webhookUrl` - (Optional) Endpoint for notifications

📖 [Full manifest documentation](https://miniapps.farcaster.xyz/reference/manifest)

---

### 2. Install the Farcaster SDK

The SDK helps your app interact with Farcaster features like user data and wallets.

```bash
npm install @farcaster/frame-sdk
```

Or with yarn:
```bash
yarn add @farcaster/frame-sdk
```

---

### 3. Initialize the SDK in Your App

Create a component or hook to initialize the SDK. Here's a simple example:

**Create `src/hooks/useFarcaster.ts`:**

```typescript
'use client';

import { useEffect, useState } from 'react';
import sdk from '@farcaster/frame-sdk';

export function useFarcaster() {
  const [isSDKLoaded, setIsSDKLoaded] = useState(false);
  const [context, setContext] = useState<any>(null);

  useEffect(() => {
    const load = async () => {
      // Load the SDK context
      const context = await sdk.context;
      setContext(context);
      setIsSDKLoaded(true);
      
      // Tell Farcaster your app is ready
      sdk.actions.ready();
    };
    
    load();
  }, []);

  return { isSDKLoaded, context };
}
```

**Use it in your page component:**

```typescript
'use client';

import { useFarcaster } from '@/hooks/useFarcaster';

export default function Home() {
  const { isSDKLoaded, context } = useFarcaster();

  if (!isSDKLoaded) {
    return <div>Loading...</div>;
  }

  return (
    <div>
      <h1>Welcome to my Farcaster Mini App!</h1>
      {context?.user && (
        <p>Hello, {context.user.username}!</p>
      )}
    </div>
  );
}
```

📖 [SDK Context documentation](https://miniapps.farcaster.xyz/sdk/context)

---

### 4. Key Features You Can Use

#### **Get User Information**
```typescript
const { user } = await sdk.context;
console.log(user.fid); // User's Farcaster ID
console.log(user.username); // User's username
console.log(user.displayName); // User's display name
console.log(user.pfpUrl); // Profile picture URL
```

#### **Open Ethereum Wallet**
```typescript
import sdk from '@farcaster/frame-sdk';

// Send a transaction
const result = await sdk.wallet.ethProvider.request({
  method: 'eth_sendTransaction',
  params: [{
    to: '0x...',
    value: '0x...',
  }],
});
```
📖 [Wallet documentation](https://miniapps.farcaster.xyz/sdk/ethereum-wallet)

#### **Share Your App**
```typescript
// Let users share your app with their followers
sdk.actions.openUrl('https://warpcast.com/~/compose?text=Check%20out%20this%20app!%20https://yourapp.com');
```
📖 [Sharing guide](https://miniapps.farcaster.xyz/guides/sharing)

#### **Trigger Haptics (Vibration)**
```typescript
// Add tactile feedback
sdk.actions.haptics('impact_light');
// Options: 'impact_light', 'impact_medium', 'impact_heavy', 'success', 'warning', 'error'
```
📖 [Haptics documentation](https://miniapps.farcaster.xyz/sdk/haptics)

#### **Close the Mini App**
```typescript
// Close your app programmatically
sdk.actions.close();
```

---

### 5. Testing Your Mini App

1. **Deploy your app** to a public URL (Vercel, Railway, etc.)
2. Make sure `/.well-known/farcaster.json` is accessible
3. **Test the manifest:** Visit `https://yourapp.com/.well-known/farcaster.json`
4. **Submit for testing** using Warpcast developer tools

📖 [Getting Started guide](https://miniapps.farcaster.xyz/introduction/getting-started)

---

### 6. Publishing Your App

Once everything works:

1. Ensure your manifest is properly configured
2. Test all functionality
3. Submit your app for review
4. Once approved, users can discover it in the Farcaster app directory

📖 [Publishing guide](https://miniapps.farcaster.xyz/guides/publishing)

---

## Common Next.js Configuration Tips

### Make `.well-known` accessible

Next.js automatically serves files from the `public` folder, so:
```
public/.well-known/farcaster.json → https://yourapp.com/.well-known/farcaster.json
```

### Use Client Components for SDK

The Farcaster SDK runs in the browser, so use `'use client'` directive:

```typescript
'use client';

import sdk from '@farcaster/frame-sdk';
```

### Environment Variables

Store your configuration in `.env.local`:

```bash
NEXT_PUBLIC_APP_URL=https://yourapp.com
```

---

## Helpful Resources

- 📘 [Official Farcaster Mini Apps Documentation](https://miniapps.farcaster.xyz/)
- 🎯 [Getting Started Guide](https://miniapps.farcaster.xyz/introduction/getting-started)
- 🛠️ [SDK Reference](https://miniapps.farcaster.xyz/sdk/context)
- 📦 [Example Mini Apps](https://miniapps.farcaster.xyz/examples)
- ❓ [FAQ](https://miniapps.farcaster.xyz/faq)
- 💬 [Join the Farcaster Developer Community](https://warpcast.com/~/channel/fc-devs)

---

## Quick Checklist

- [ ] Create `.well-known/farcaster.json` manifest file
- [ ] Add app icon (200x200px minimum)
- [ ] Add splash image (1500x1500px recommended)
- [ ] Install `@farcaster/frame-sdk`
- [ ] Initialize SDK with `sdk.context` and `sdk.actions.ready()`
- [ ] Test that manifest is accessible at `/.well-known/farcaster.json`
- [ ] Deploy to a public URL
- [ ] Test your app in Farcaster
- [ ] Submit for publishing

---

## Need Help?

- Check the [FAQ](https://miniapps.farcaster.xyz/faq)
- Read the [AI/LLM Guidelines](https://miniapps.farcaster.xyz/llms-txt)
- Ask in the [Farcaster Developers Channel](https://warpcast.com/~/channel/fc-devs)

**Good luck building your Farcaster Mini App! 🚀**

