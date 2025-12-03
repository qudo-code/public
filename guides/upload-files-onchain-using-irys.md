# Upload Files On-Chain Using Irys
![](https://raw.githubusercontent.com/qudo-code/public/refs/heads/main/public/media/Pasted%20image%2020251202165342.png)
In this guide, we will learn how to upload files on-chain using Solana, Irys, and TypeScript.
- [Solana](https://solana.com/): A fast and cheap decentralized blockchain for payments.
- [Irys](https://irys.xyz/): A fast and cheap decentralized blockchain for storage.
## System Requirements
To follow this guide, you will need to have the following installed.

[Install Bun](https://bun.sh/) (or Node.js)
```bash
curl -fsSL https://bun.sh/install | bash
```
[Install Solana CLI](https://docs.solana.com/cli/install-solana-cli-tools)
```bash
curl --proto '=https' --tlsv1.2 -sSfL https://solana-install.solana.workers.dev | bash
```
## Setup Project
Open your terminal and navigate to the directory where you want to create your project and initialize a new project.
```bash
mkdir my-project
cd my-project
bun init
```
### Add Dependencies
Once initialized, add the following dependencies to your project.
```bash
bun add @solana/web3.js @irys/sdk @irys/upload-solana
```
### Update Scripts
Open the `package.json` that was created during initialization and add a `scripts` block with the following.
```json
{
  "scripts": {
    "upload": "bun run index.ts",
    "fund": "bun run index.ts fund",
    "keypair": "solana-keygen new --outfile ./.env.keypair",
  }
}
```

### Generate Keypair
Above we added a script that will generate a new keypair for us and write it to the file `./.env.keypair`.

_**Warning**: Running the script again will overwrite any existing keypair._

```bash
bun keypair
```
### Get Devnet Solana Tokens
We need SOL in our wallet to get started. For development purposes, you can get free devnet SOL from [76 Dev Discord](https://discord.gg/76devs), [Solana Devnet Faucet](https://faucet.solana.com/) or you can try running the following command.

```bash
solana airdrop 2 --keypair ./.env.keypair
```
_**Note**: While devnet is awesome for testing, transactions and uploads won't be permanent._

### Add Upload Logic
Create an `index.ts` at the root of your project and copy/paste the following.
```ts
import { Uploader } from "@irys/upload";
import { Solana } from "@irys/upload-solana";
import fs from "fs";

const config = {
  // Update this RPC url to a mainnet RPC when you're ready
  // (https://app.extrnode.com/)
  rpc: "https://api.devnet.solana.com",
  keypair: "./.env.keypair",
  uploadsDirectory: "./uploads",
  outputDirectory: "./output",
};

const privateKey = JSON.parse(fs.readFileSync(config.keypair, "utf8"));

const getIrysUploader = async () => {
  const irysUploader = await Uploader(Solana).withWallet(privateKey).withRpc(config.rpc);
  return irysUploader;
};

const fundIrysWallet = async (amount: number) => {
  const irysUploader = await getIrysUploader();
  const fundTx = await irysUploader.fund(irysUploader.utils.toAtomic(amount));
}

const uploadFolder = async () => {
  // Will output to uploads-manifest.json and
  // uploads-manifest.csv in current directory
  console.log("Uploading files...");
  console.log(`☁️ Upload Directory: ${config.uploadsDirectory}\n`);

  const irysUploader = await getIrysUploader();

  const receipt = await irysUploader.uploadFolder(config.uploadsDirectory, {
    indexFile: "", // Optional index file (file the user will load when accessing the manifest)
    batchSize: 50, // Number of items to upload at once
    keepDeleted: false, // whether to keep now deleted items from previous uploads
  }); // Returns the manifest ID

  console.log(`Files uploaded. Manifest ID ${receipt?.id}`);

  fs.writeFileSync("uploads-manifest.json", JSON.stringify(receipt, null, 2));
}

if(process.argv[2] === "fund") {
  fundIrysWallet(0.005);
} else if(process.argv[2] === "upload") {
  uploadFolder();
}
````
## Upload Directory
### Funding Irys Account
If this is your first upload or you've done a few uploads, it's probably time to top off your storage account using your Solana wallet. You can do so by running the `bun fund` script we made earlier.

### Start Upload
Create a new folder called `uploads` in the root of your project and place files to be uploaded. Run the following script to upload using the config in `index.ts`.
```bash
bun upload
````
