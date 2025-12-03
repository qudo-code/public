
# Upload Folder to Irys
```ts
// to use this script"
// via the solana cli run "solana-keygen new -o upload.json"
// fund the solana wallet
// fund the irys wallet by uncommenting the fund line
// upload the folder by uncommenting the upload line

import { Uploader } from "@irys/upload";
import { Solana } from "@irys/upload-solana";
import fs from "fs";

const config = {
  rpc: "https://api.mainnet-beta.solana.com",
  keypair: "./upload.json",
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
}

// Use as needed
// fundIrysWallet(0.005);
// uploadFolder();
```
