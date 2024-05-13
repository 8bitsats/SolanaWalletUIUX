---
description: >-
  This guide will walk you through the process of integrating the Solana Wallet
  Adapter in a Next.js application, enabling you to use Solana wallets
  seamlessly.
---

# Integrating Solana Wallet Adapter in a Next.js Application

#### Table of Contents

1. Creating a New Next.js Project
2. Installing Solana Wallet Adapter Dependencies
3. Setting Up Solana Wallet Adapter in Your Next.js App
4. Adding a Connect Wallet Button
5. Utilizing the Wallet Context in Other Next.js Pages

#### Creating a New Next.js Project

First, create a new Next.js project if you don't already have one. Run the following command in your terminal:

```
npx create-next-app@latest my-solana-app
cd my-solana-app
```

#### Installing Solana Wallet Adapter Dependencies

Next, install the necessary Solana wallet adapter dependencies. Run the following command:

```
npm install @solana/wallet-adapter-react @solana/wallet-adapter-base @solana/wallet-adapter-react-ui @solana/wallet-adapter-wallets @solana/wallet-adapter-react-ui @solana/wallet-adapter-walletconnect @solana/web3.js
```

#### Setting Up Solana Wallet Adapter in Your Next.js App

Create a new file named `wallet.js` inside the `src` directory (or the appropriate directory based on your project structure). This file will set up the Solana Wallet Adapter context.

```
// src/wallet.js
import React, { useMemo } from 'react';
import { ConnectionProvider, WalletProvider } from '@solana/wallet-adapter-react';
import { WalletModalProvider } from '@solana/wallet-adapter-react-ui';
import { PhantomWalletAdapter } from '@solana/wallet-adapter-wallets';
import { clusterApiUrl } from '@solana/web3.js';

const WalletContextProvider = ({ children }) => {
    const network = clusterApiUrl('mainnet-beta');
    const wallets = useMemo(() => [new PhantomWalletAdapter()], []);

    return (
        <ConnectionProvider endpoint={network}>
            <WalletProvider wallets={wallets} autoConnect>
                <WalletModalProvider>{children}</WalletModalProvider>
            </WalletProvider>
        </ConnectionProvider>
    );
};

export default WalletContextProvider;
```

In `pages/_app.js`, wrap your application with the `WalletContextProvider`.

```
// pages/_app.js
import '../styles/globals.css';
import WalletContextProvider from '../src/wallet';

function MyApp({ Component, pageProps }) {
    return (
        <WalletContextProvider>
            <Component {...pageProps} />
        </WalletContextProvider>
    );
}

export default MyApp;
```

#### Adding a Connect Wallet Button

Create a new component for the connect wallet button, for example, `components/ConnectWalletButton.js`.

```
// components/ConnectWalletButton.js
import { useWallet } from '@solana/wallet-adapter-react';
import { WalletMultiButton } from '@solana/wallet-adapter-react-ui';

const ConnectWalletButton = () => {
    const { connected } = useWallet();

    return (
        <div>
            {connected ? <p>Wallet Connected</p> : <WalletMultiButton />}
        </div>
    );
};

export default ConnectWalletButton;
```

Use this component in any page, such as `pages/index.js`.

```
// pages/index.js
import ConnectWalletButton from '../components/ConnectWalletButton';

export default function Home() {
    return (
        <div>
            <h1>Welcome to My Solana App</h1>
            <ConnectWalletButton />
        </div>
    );
}
```

#### Utilizing the Wallet Context in Other Next.js Pages

You can access the wallet context using the `useConnection` and `useWallet` hooks provided by the Solana Wallet Adapter. Here’s an example of how to use these hooks in another page:

```
// pages/nft.js
import { useConnection, useWallet } from '@solana/wallet-adapter-react';
import { useEffect, useState } from 'react';
import { getParsedNftAccountsByOwner } from '@nfteyez/sol-rayz';

export default function NftPage() {
    const { connection } = useConnection();
    const { publicKey } = useWallet();
    const [nfts, setNfts] = useState([]);

    useEffect(() => {
        if (publicKey) {
            getParsedNftAccountsByOwner({ publicAddress: publicKey.toString(), connection })
                .then((nftAccounts) => setNfts(nftAccounts))
                .catch((err) => console.error(err));
        }
    }, [publicKey, connection]);

    return (
        <div>
            <h1>Your NFTs</h1>
            {nfts.length > 0 ? (
                nfts.map((nft, index) => <div key={index}>{nft.data.name}</div>)
            ) : (
                <p>No NFTs found</p>
            )}
        </div>
    );
}
```
