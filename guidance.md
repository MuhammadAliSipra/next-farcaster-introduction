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
    "header": "eyJmaWQiOjEsInR5cGUiOiJjdXN0b2R5Iiwia2V5IjoiMHgwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwIn0",
    "payload": "eyJkb21haW4iOiJleGFtcGxlLmNvbSJ9",
    "signature": "MHg..."
  },
  "frame": {
    "version": "1",
    "name": "Your App Name",
    "iconUrl": "https://yourdomain.com/icon.png",
    "homeUrl": "https://yourdomain.com",
    "splashImageUrl": "https://yourdomain.com/splash.png",
    "splashBackgroundColor": "#ffffff"
  }
}
```

**⚠️ Note on `accountAssociation`:**
The `accountAssociation` field is used to verify your domain ownership. For testing purposes, you can omit this entire section initially. To properly set it up for production, you'll need to:
1. Generate a signature proving you own the Farcaster account
2. Use Farcaster's account association tools
3. For now, you can remove the entire `accountAssociation` object and your app will still work

**Minimal working example for testing:**
```json
{
  "frame": {
    "version": "1",
    "name": "Your App Name",
    "iconUrl": "https://yourdomain.com/icon.png",
    "homeUrl": "https://yourdomain.com",
    "splashImageUrl": "https://yourdomain.com/splash.png",
    "splashBackgroundColor": "#ffffff"
  }
}
```

**Important fields to customize:**
- `version` - Always use "1" (required)
- `name` - Your app's display name, max 20 characters (required)
- `iconUrl` - Square app icon, at least 200x200px PNG or JPG (required)
- `homeUrl` - The URL where your app lives (required)
- `splashImageUrl` - Loading screen image, 1500x1500px recommended (required)
- `splashBackgroundColor` - Hex color for the loading screen background (optional)
- `webhookUrl` - Endpoint for receiving notifications (optional, omitted above)

📖 More info: https://miniapps.farcaster.xyz/

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

📖 Learn more: https://miniapps.farcaster.xyz/

---

### 3.5. Detecting Farcaster Environment (Optional but Recommended)

It's good practice to check if your app is running inside Farcaster or in a regular browser:

```typescript
'use client';

import { useEffect, useState } from 'react';
import sdk from '@farcaster/frame-sdk';

export function useFarcaster() {
  const [isSDKLoaded, setIsSDKLoaded] = useState(false);
  const [context, setContext] = useState<any>(null);
  const [isInFarcaster, setIsInFarcaster] = useState(false);

  useEffect(() => {
    const load = async () => {
      try {
        // Check if running in Farcaster environment
        const inFarcaster = sdk.context !== null;
        setIsInFarcaster(inFarcaster);
        
        if (inFarcaster) {
          const context = await sdk.context;
          setContext(context);
          sdk.actions.ready();
        }
      } catch (error) {
        console.error('Failed to load Farcaster SDK:', error);
      } finally {
        setIsSDKLoaded(true);
      }
    };
    
    load();
  }, []);

  return { isSDKLoaded, context, isInFarcaster };
}
```

**Use it to show different UI:**

```typescript
export default function Home() {
  const { isSDKLoaded, context, isInFarcaster } = useFarcaster();

  if (!isSDKLoaded) {
    return <div>Loading...</div>;
  }

  if (!isInFarcaster) {
    return (
      <div>
        <h1>Please open this app in Warpcast</h1>
        <p>This app is designed to run as a Farcaster Mini App.</p>
      </div>
    );
  }

  return (
    <div>
      <h1>Welcome, {context.user.displayName}!</h1>
      <p>You're viewing this inside Farcaster</p>
    </div>
  );
}
```

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

#### **Share Your App**
```typescript
// Let users share your app with their followers
sdk.actions.openUrl('https://warpcast.com/~/compose?text=Check%20out%20this%20app!%20https://yourapp.com');
```

#### **Trigger Haptics (Vibration)**
```typescript
// Add tactile feedback
sdk.actions.haptics('impact_light');
// Options: 'impact_light', 'impact_medium', 'impact_heavy', 'success', 'warning', 'error'
```

#### **Close the Mini App**
```typescript
// Close your app programmatically
sdk.actions.close();
```

#### **Open External URLs**
```typescript
// Open a URL in the user's browser
sdk.actions.openUrl('https://example.com');
```

#### **Add to Home Screen Prompt**
```typescript
// Prompt user to add your app to home screen
sdk.actions.addFrame();
```

---

### 4.5. Understanding SDK Types (for TypeScript users)

Here are the key types you'll work with:

```typescript
// User Context Type
interface User {
  fid: number;                    // Farcaster ID
  username: string;                // @username
  displayName: string;             // Display name
  pfpUrl: string;                  // Profile picture URL
}

// Full Context Type
interface Context {
  user: User;
  location: {
    type: 'launch' | 'cast' | 'composer';
  };
  client: {
    clientFid: string;
    added: boolean;               // Is app added to home screen?
  };
}

// Available SDK Actions
sdk.actions.ready()               // Tell Farcaster app is ready
sdk.actions.close()               // Close the mini app
sdk.actions.openUrl(url: string)  // Open external URL
sdk.actions.addFrame()            // Prompt to add to home screen
sdk.actions.haptics(type: 'impact_light' | 'impact_medium' | 'impact_heavy' | 'success' | 'warning' | 'error')

// Available SDK Properties
sdk.context                       // Promise<Context>
sdk.wallet.ethProvider           // Ethereum wallet provider
```

---

### 5. Testing Your Mini App

**Step-by-step testing:**

1. **Deploy your app** to a public URL (Vercel, Railway, etc.)
   
2. **Verify the manifest is accessible:**
   - Visit `https://yourapp.com/.well-known/farcaster.json` in your browser
   - You should see your JSON manifest displayed
   - If you get a 404 error, check that the file is in `public/.well-known/farcaster.json`

3. **Test in Warpcast (the official Farcaster client):**
   - Open Warpcast app on your mobile device
   - Navigate to your mini app URL or use a test frame
   - The SDK should load and `sdk.actions.ready()` should be called

4. **Debug tips:**
   - Check browser console for SDK errors
   - Verify all required manifest fields are present
   - Ensure your icon and splash images are accessible
   - Test that `sdk.context` returns user information

📖 Full documentation: https://miniapps.farcaster.xyz/

---

### 6. Publishing Your App

Once everything works locally:

1. **Double-check your manifest:**
   - All required fields are filled
   - Images are properly hosted and accessible
   - URLs are production URLs (not localhost)

2. **Test thoroughly:**
   - All user interactions work
   - Wallet transactions work (if applicable)
   - App loads properly on mobile

3. **Submit your app:**
   - Your app will be discoverable in the Farcaster ecosystem
   - Users can add it to their home screens
   - It appears in the Mini App directory

📖 More details: https://miniapps.farcaster.xyz/

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

### Deployment on Vercel (Recommended)

Vercel is the easiest way to deploy Next.js apps:

1. **Push your code to GitHub**
   ```bash
   git init
   git add .
   git commit -m "Initial commit"
   git remote add origin YOUR_REPO_URL
   git push -u origin main
   ```

2. **Deploy on Vercel:**
   - Go to https://vercel.com
   - Click "Import Project"
   - Select your GitHub repository
   - Vercel auto-detects Next.js settings
   - Click "Deploy"

3. **Update your manifest:**
   - Once deployed, get your production URL (e.g., `yourapp.vercel.app`)
   - Update `homeUrl` in `.well-known/farcaster.json`
   - Update `iconUrl` and `splashImageUrl` with full URLs
   - Commit and push changes
   - Vercel will auto-deploy

4. **Verify:**
   - Visit `https://yourapp.vercel.app/.well-known/farcaster.json`
   - Should return your manifest JSON

**Alternative platforms:**
- **Railway:** https://railway.app (good for apps with backends)
- **Netlify:** https://netlify.com (similar to Vercel)
- **Cloudflare Pages:** https://pages.cloudflare.com (fastest CDN)

---

## Helpful Resources

- 📘 **Official Documentation:** https://miniapps.farcaster.xyz/
- 📦 **NPM Package:** https://www.npmjs.com/package/@farcaster/frame-sdk
- 💬 **Developer Community:** Join the fc-devs channel on Warpcast
- 🐙 **GitHub Examples:** Search for "farcaster mini app" examples on GitHub

---

## Quick Checklist

- [ ] Create `.well-known/farcaster.json` manifest file in `public/` folder
- [ ] Add app icon (minimum 200x200px, PNG or JPG)
- [ ] Add splash image (1500x1500px recommended, PNG or JPG)
- [ ] Install `@farcaster/frame-sdk` package
- [ ] Initialize SDK in a client component with `sdk.context` and `sdk.actions.ready()`
- [ ] Verify manifest is accessible at `https://yourapp.com/.well-known/farcaster.json`
- [ ] Deploy your app to a public URL (Vercel, Railway, etc.)
- [ ] Test the app in Warpcast mobile app
- [ ] Verify user authentication works (check `context.user`)
- [ ] Test any wallet interactions (if applicable)
- [ ] Submit your app for discovery

---

## Common Issues & Solutions

### Issue 1: Manifest returns 404
**Solution:** Ensure the file is at `public/.well-known/farcaster.json`. Next.js serves files from `public/` at the root URL.

### Issue 2: SDK not loading
**Solution:** Make sure you're using `'use client'` directive at the top of your component file. The SDK only works in client-side components.

### Issue 3: User context is undefined
**Solution:** Wait for the SDK to load completely. Use the loading state pattern shown in the examples above.

### Issue 4: Images not loading in manifest
**Solution:** Use full absolute URLs (including `https://`) for all image paths in your manifest.

### Issue 5: App works locally but not in Warpcast
**Solution:** Ensure you're using production URLs in your manifest, not `localhost`. Deploy your app first, then test.

---

## Need Help?

- 📖 **Main Documentation:** https://miniapps.farcaster.xyz/
- 🔍 **Search the docs** for specific features and APIs
- 💬 **Ask the community** in Warpcast's fc-devs channel
- 🐛 **Check GitHub Issues** for the frame-sdk repository

---

## Example: Real-World Mini App Structure

Here's what your final Next.js project should look like:

```
your-app/
├── public/
│   ├── .well-known/
│   │   └── farcaster.json          # ← Manifest file
│   ├── icon.png                     # ← App icon (200x200+)
│   └── splash.png                   # ← Splash screen (1500x1500)
├── src/
│   ├── app/
│   │   ├── page.tsx                 # ← Main app (client component)
│   │   └── layout.tsx
│   └── hooks/
│       └── useFarcaster.ts          # ← SDK initialization hook
├── package.json
└── next.config.js
```

**Good luck building your Farcaster Mini App! 🚀**

---

*Last updated: November 2025 | Based on @farcaster/frame-sdk*


