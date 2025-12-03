# On-Chain User Profiles
![](https://raw.githubusercontent.com/qudo-code/public/refs/heads/main/media/Pasted%20image%2020251203121607.png)
A user profiles protocol stored on Arweave, verified on Solana via compressed NFTs.
https://github.com/qudo-code/eternal-users-protocol

## React Usage

### Prerequisites
Ensure your app has Solana wallet adapter and required providers. EUP will utilize the `useWallet` hook.
https://github.com/anza-xyz/wallet-adapter
```jsx
function App() {
    return (
        <ConnectionProvider endpoint={endpoint}>
          <WalletProvider wallets={[]}>
              <WalletModalProvider>
                <OtherComponent />
              </WalletModalProvider>
          </WalletProvider>
        </ConnectionProvider>
    )
}
```

### Create/Update User
Create or update a user profile via the `update` method. This will create a profile if one doesn't already exist or update existing. This flow consists of uploading metadata to Arweave then minting or updating the users profile NFT.
_Expect about three wallet popups (fund upload, sign upload, mint/update NFT)_

```jsx
import { useEup } from "eternal-users-protocol";

import { ConnectionProvider, WalletProvider } from '@solana/wallet-adapter-react';
import { WalletAdapterNetwork } from '@solana/wallet-adapter-base';
import { WalletModalProvider, WalletMultiButton } from '@solana/wallet-adapter-react-ui';
import { clusterApiUrl } from '@solana/web3.js';

async function OtherComponent() {
    // The network can be set to 'devnet', 'testnet', or 'mainnet-beta'.
    const network = WalletAdapterNetwork.Devnet;

    // You can also provide a custom RPC endpoint.
    const endpoint = useMemo(() => clusterApiUrl(network), [network]);

    // Initialize EUP
    const eup = useEup(endpoint);

    return (
        <ConnectionProvider endpoint={endpoint}>
            <WalletProvider wallets={[]}>
                <WalletModalProvider>
                    <WalletMultiButton />
                    <input type="text" onChange={eup.set.name}>
                    <input type="file" onChange={eup.set.image}>
                    <button onClick={eup.update}>Update</button>
                </WalletModalProvider>
            </WalletProvider>
        </ConnectionProvider>
    )
}
```
### Values
_Read only values_
#### `eup.draft`
Get the current pending/drafted details.
#### `eup.value`
Get the current on-chain details.
#### `eup.state`
Get loading state ("resting", "uploading", "updating", "error")

### Methods
#### `eup.set.name`, `eup.set.image`
Setters for updating the`draft` state.
#### `eup.save`
Save the drafted user details on-chain.
#### `eup.add.item`
Associate a piece of metadata to a user such as a badge or achievement. 
#### `eup.reset`
Clear drafted user details and revert to existing on-chain deatils.
