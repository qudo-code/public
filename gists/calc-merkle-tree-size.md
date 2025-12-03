# Calc Merkle Tree Size
```ts
import {
  ALL_DEPTH_SIZE_PAIRS,
  getConcurrentMerkleTreeAccountSize,
} from "@solana/spl-account-compression";
import { Connection, LAMPORTS_PER_SOL } from "@solana/web3.js";

type CostResult = {
  size: number;
  cost: number;
};

type TreeResult = {
  maxProof: number;
  canopy: number;
  maxBuffer: number;
  maxDepth: number;
  supply: number;
} & CostResult;

// Get cost to create tree
const getTreeCost = async (size: number, connection: Connection) => {
  try {
    const rent =
      (await connection.getMinimumBalanceForRentExemption(size)) /
      LAMPORTS_PER_SOL;

    return rent;
  } catch (error) {}

  return 0;
};

// Accepts a supply as a number and returns a tree that should work.
export const calculateTreeSize = async (
  supply: number
): Promise<TreeResult | null> => {
  // Tensor max proof is 9 so this should be supported and provide extra room.
  const MAX_PROOF_SIZE = 8;

  // Include results with the following max buffer size.
  const MAX_BUFFER_SIZES = [64];

  const connection = new Connection("https://api.mainnet-beta.solana.com");

  const relevantDepthPairs = ALL_DEPTH_SIZE_PAIRS.filter(
    ({ maxBufferSize }) => MAX_BUFFER_SIZES.indexOf(maxBufferSize) >= 0
  );

  let result: TreeResult | null = null;

  for (const pair of relevantDepthPairs) {
    const canopy = pair.maxDepth - MAX_PROOF_SIZE;

    const size = getConcurrentMerkleTreeAccountSize(
      pair.maxDepth,
      pair.maxBufferSize,
      canopy
    );

    const cost = await getTreeCost(size, connection);

    const maxSupply = Math.pow(2, pair.maxDepth);

    // If no cost or max buffer isn't one we are looking for.
    if (!cost || maxSupply <= supply) {
      continue;
    }

    result = {
      maxProof: MAX_PROOF_SIZE,
      canopy,
      maxBuffer: pair.maxBufferSize,
      maxDepth: pair.maxDepth,
      size,
      cost,
      supply: maxSupply,
    };

    break;
  }

  return result;
};

```
