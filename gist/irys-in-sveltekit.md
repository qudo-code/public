
# Irys in SvelteKit
Upload files on-chain using Irys and Sveltekit.
`+layout.svelte`
```html
<script lang="ts">
    import {
      WalletModal,
      getShowConnectWallet,
      getWallets,
      hideConnectWallet,
      init,
      localStorageKey,
      network,
      walletStore
    } from "$lib/wallets";

    import {
      ConnectionProvider,
      WalletProvider,
    } from "@aztemi/svelte-on-solana-wallet-adapter-ui";

    import { onMount } from "svelte";
    
    let { children } = $props();
    let showConnectWallet = $derived(getShowConnectWallet());
    
    const wallets = $derived.by(getWallets);
    
    const onConnect = async (event: { detail: string }) => {
        await $walletStore.select(event.detail);
        await $walletStore.connect();
        hideConnectWallet();
    };
    
    onMount(() => {
        init();
    });
</script>

<ConnectionProvider {network} />
<WalletProvider {localStorageKey} {wallets} autoConnect={true} />

{#if showConnectWallet}
    <WalletModal
        maxNumberOfWallets={20}
        on:close={hideConnectWallet}
        on:connect={onConnect}
    />
{/if}

{@render children()}
```

`irys.ts`
```ts
import { WebUploader } from "@irys/web-upload";
import { WebSolana } from "@irys/web-upload-solana";
import type { Irys, UploadInput, UploadResponse } from "./types";

export const irysGateway = (dev: boolean) =>
  dev ? "https://devnet.irys.xyz" : "https://gateway.irys.xyz";

export const fileToDataUri = (file?: File): Promise<string | null> =>
  new Promise((resolve, reject) => {
    if (!file) return reject("No file selected");

    const reader = new FileReader();
    reader.onload = (e) => {
      resolve(e.target?.result as string);
    };
    reader.readAsDataURL(file);
  });

export const bufferFromFile = (file: File): Promise<Buffer> =>
  new Promise((resolve, reject) => {
    if (!file) return reject("No file selected");

    const reader = new FileReader();

    reader.readAsArrayBuffer(file);
    reader.onload = async (event) => {
      const arrayBuffer = event.target?.result as ArrayBuffer;
      const uint8Array = new Uint8Array(arrayBuffer);
      resolve(Buffer.from(uint8Array));
    };
  });

export const getIrys = async (
  adapter: any,
  rpcUrl: string,
  devnet: boolean
) => {
  try {
    let irysUploader;
    if (devnet) {
      irysUploader = await WebUploader(WebSolana)
        .withProvider(adapter)
        .withRpc(rpcUrl)
        .devnet();
    } else {
      irysUploader = await WebUploader(WebSolana).withProvider(adapter);
    }

    return irysUploader;
  } catch (error) {
    console.error("Error connecting to Irys:", error);
    throw new Error("Error connecting to Irys");
  }
};

export const fundIrysBalance = async (amount: number, irys: Irys) => {
  const fundTx = await irys.fund(irys.utils.toAtomic(amount));
  const response = await irys.funder.submitFundTransaction(fundTx.id);

  return response;
};

export const getIrysBalance = async (irys: Irys) => {
  // Get loaded balance in atomic units
  const atomicBalance = await irys.getBalance();
  // Convert balance to standard
  const convertedBalance = irys.utils.fromAtomic(atomicBalance);

  return Number(convertedBalance);
};

export const uploadToIrys = async (
  input: UploadInput,
  irys: Irys
): Promise<UploadResponse> => {
  const buffer = await bufferFromFile(input.file);
  const upload = await irys.upload(buffer, {
    tags: [
      {
        name: "sol-directory-media",
        value: input.type,
      },
    ],
  });

  return {
    id: upload.id,
    uri: `${irysGateway(input.dev)}/${upload.id}`,
    upload_transaction: upload.signature,
    formatting: input.type,
    type: input.type,
  };
};
```

`media.svelte`
```ts
<script lang="ts">
    import { dev } from "$app/environment";
    import Button from "$lib/components/ui/button/button.svelte";
    import * as Tabs from "$lib/components/ui/tabs/index.js";
    import { walletStore } from "$lib/wallets";
    import ConnectWallet from "$lib/wallets/connect.svelte";
    import type { IrysMedia, Media } from "@repo/sdk";
    import { CheckIcon, Images, Link, Loader2, Loader2Icon, PlusCircle, Upload, User, Wallet } from "lucide-svelte";
    import type { ReadableQuery } from "svelte-apollo";
    import { toast } from "svelte-sonner";
    import { tweened } from "svelte/motion";
    import Input from "../ui/input/input.svelte";
    import Item from "./item.svelte";
    import { media } from "./media.svelte";
    import SelectFile from "./select-file.svelte";

    const animatedBalance = tweened(0, { duration: 1000 });

    const initMedia = async () => {
      await media.init($walletStore);
      await media.getMedia();
      await media.getBalance();
    }

    // Initialize UserMedia when wallet is connected/changed
    $effect(() => {
      if ($walletStore?.publicKey) initMedia();
    });

    let needsFunding = $derived(typeof media.balance.data === "number" && media.balance.data < 0.0003);
    
  
    let selectedMedia = $state<IrysMedia | null>(null);
    let showAllMedia = $state(false);

    let mediaItems = $derived((showAllMedia ? media.media.data : media.media.data?.slice(0, 6) || []) as IrysMedia[]);

    const handleUpload = async () => {
      if (!media.selectedFile.data?.file) return;
      toast(`Preparing upload transaction, check wallet...`);
      await media.uploadMedia({
        file: media.selectedFile.data.file,
        type: "avatar",
        formatting: "",
      });
      toast(`Upload success!`);
      await media.getMedia();
    }

    const handleFund = async () => {
      try {
        const amount = 0.005;
        toast(`Preparing funding transaction, check wallet...`);
        await media.fundBalance(amount);
        toast(`Success! Added ${amount} SOL to your account.`);
      } catch (error) {
        console.error(error);
        toast(`Error adding funds, try again.`);
      }
    }

    const handleSetAvatar = async () => {
      try {
        if(!selectedMedia?.uri) return;
        await media.setAvatar(selectedMedia?.id);
        toast(`Success! Your avatar has been updated.`);
      } catch (error) {
        console.error(error);
        toast(`Error setting avatar, try again.`);
      }
    }
</script>
```

`media.svelte.ts`
```ts
import { dev } from "$app/environment";
import { PUBLIC_API_URL, PUBLIC_RPC_URL } from "$env/static/public";
import type {
  Irys,
  IrysMedia,
  MediaType,
  State,
  UploadResponse,
} from "./types";

import {
  irysGateway,
  fundIrysBalance,
  getIrys,
  getIrysBalance,
  uploadToIrys,
} from "./util";

class UserMedia {
  public irys: Irys | null = null;
  public ready: boolean = false;
  public showMediaManager: boolean = $state(false);
  public selectedFile: State<{
    file: File | null;
    dataUri: string | null;
  }> = $state({
    data: {
      file: null,
      dataUri: null,
    },
    error: null,
    loading: false,
    success: false,
  });
  public media: State<IrysMedia[]> = $state({
    data: [],
    error: null,
    loading: false,
    success: false,
  });
  public upload: State<UploadResponse> = $state({
    data: null,
    error: null,
    loading: false,
    success: false,
  });
  public balance: State<number | null> = $state({
    data: null,
    error: null,
    loading: false,
    success: false,
  });
  public fund: State<string> = $state({
    data: "",
    error: null,
    loading: false,
    success: false,
  });

  constructor() {}

  // Accepts $walletStore from Solana Svelte Wallet adapter
  public async init(adapter: any) {
    const irys = await getIrys(adapter, PUBLIC_RPC_URL, dev);
    this.irys = irys;
    this.ready = true;
  }

  public async getMedia() {
    try {
      this.media.loading = true;
      const getMediaQuery = `
        query getMedia {
          transactions(
            owners: ["${this.irys?.address}"]
            tags: [{ name: "sol-directory-media", values: ["avatar"] }]
          ) {
            edges {
              node {
                id
                address
              }
            }
          }
        }
      `;

      const response = await fetch(irysGateway(dev) + "/graphql", {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify({ query: getMediaQuery }),
      });

      const data = await response.json();

      const formattedMedia = data.data.transactions.edges
        .map(({ node }: { node: { id: string } }) => ({
          id: node.id,
          uri: `${irysGateway(dev)}/${node.id}`,
        }))
        .reverse();

      this.media.data = formattedMedia ?? [];
      this.media.success = true;
    } catch (error) {
      this.media.error = error as Error;
      console.error("Error fetching user media:", error);
    } finally {
      this.media.loading = false;
    }
  }

  public async getBalance() {
    this.balance.loading = true;
    try {
      if (!this.irys) throw new Error("Irys not initialized");

      const balance = await getIrysBalance(this.irys);
      this.balance.data = balance;
      this.balance.success = true;
    } catch (error) {
      this.balance.error = error as Error;
      console.error("Error fetching balance:", error);
    } finally {
      this.balance.loading = false;
    }
  }

  // ~ $1.50 with SOL at $230
  public async fundBalance(amount: number = 0.005) {
    try {
      console.log("GOT HERE", this.irys);
      if (!this.irys) throw new Error("Irys not initialized");

      this.fund.loading = true;
      const response = await fundIrysBalance(amount, this.irys);
      console.log(response);
      this.getBalance();
      this.fund.success = true;
    } catch (error) {
      this.fund.error = error as Error;
      console.error("Error funding balance:", error);
    } finally {
      this.fund.loading = false;
    }
  }

  public async uploadMedia(input: {
    file: File;
    type: MediaType;
    formatting: string;
  }) {
    try {
      if (!this.irys) throw new Error("Irys not initialized");

      this.upload.loading = true;

      // Upload file to Irys
      const uploaded = await uploadToIrys(
        {
          file: input.file,
          type: input.type,
          formatting: input.formatting,
          dev,
        },
        this.irys
      );

      const response = await fetch(`${PUBLIC_API_URL}/user/media/create`, {
        credentials: "include",
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify({
          uri: uploaded.uri,
          onchain_id: uploaded.id,
          upload_transaction: uploaded.upload_transaction,
          formatting: input.formatting,
          type: input.type,
        }),
      });

      this.upload.data = uploaded;
      this.upload.loading = false;
      this.upload.success = true;
    } catch (error) {
      this.upload.error = error as Error;
      console.error("Error uploading media:", error);
    } finally {
      this.upload.loading = false;
    }
  }
}

export const media = new UserMedia();
```

`types.ts`
```ts
const media = {
  avatar: {},
};

import { getIrys } from "./media";

export type MediaType = keyof typeof media;

export type IrysMedia = {
  id: string;
  uri: string;
};

export type Media = {
  id: string;
  uri: string;
  user_id: string;
  type: MediaType;
  created_at: Date;
  formatting: string;
  upload_transaction: string;
};

export type UploadResponse = Omit<Media, "created_at" | "user_id">;
export type Irys = Awaited<ReturnType<typeof getIrys>>;

export type State<T> = {
  data: T | null;
  error: Error | null;
  success: boolean;
  loading: boolean;
};

export type UploadInput = {
  file: File;
  type: MediaType;
  formatting: string;
  dev: boolean;
};

export type CreateMediaResponse = {
  uri: string;
  id: string;
  formatting: string;
} | null;
```

`wallet.ts`
```ts
// Svelte 5-afied Svelte on Solana wallet adapter
// This needs to be refactored/improved since I learned a bunch of new Svelte 5 things.
import { dasApi } from "@metaplex-foundation/digital-asset-standard-api";
import { mplTokenMetadata } from "@metaplex-foundation/mpl-token-metadata";
import {
  walletAdapterIdentity,
  type WalletAdapter,
} from "@metaplex-foundation/umi-signer-wallet-adapters";
import { irysUploader } from "@metaplex-foundation/umi-uploader-irys";
import { BaseMessageSignerWalletAdapter } from "@solana/wallet-adapter-base";

// @ts-expect-error
import { walletStore } from "@aztemi/svelte-on-solana-wallet-adapter-core";

// Set supported wallets
let _wallets: BaseMessageSignerWalletAdapter[] = $state([]);
export const init = async () => {
  const { PhantomWalletAdapter } = await import(
    "@solana/wallet-adapter-wallets"
  );

  _wallets = [new PhantomWalletAdapter()];
};
export const getWallets = () => _wallets;

// Adapt walletStore to svelte 5
let _adapter = $state<{ adapter: WalletAdapter }>();
walletStore.subscribe(
  ($value: BaseMessageSignerWalletAdapter & { adapter: WalletAdapter }) => {
    _adapter = $value;
  }
);
export const getAdapter = () => _adapter;

let _showModal = $state<boolean>(false);
export const getShowConnectWallet = () => _showModal;
export const showConnectWallet = () => (_showModal = true);
export const hideConnectWallet = () => (_showModal = false);

// Create new umi instance with wallet adapter
export const createUmi = async (rpc: string) => {
  if (!_adapter) return;

  const { createUmi } = await import(
    "@metaplex-foundation/umi-bundle-defaults"
  );

  const umi = createUmi(rpc)
    .use(walletAdapterIdentity(_adapter.adapter))
    .use(mplTokenMetadata())
    .use(irysUploader())
    .use(dasApi());

  return umi;
};
```
