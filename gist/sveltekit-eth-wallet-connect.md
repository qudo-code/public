
# Connect Eth Wallet
Add "Connect Ethereum Wallet" to a SvelteKit app. 
`+layout.svelte`
```html
<script lang="ts">
	import { onMount } from 'svelte';
	import { init } from '$lib/wagmi';

	onMount(init);
</script>

<slot />
```

`nav.svelte`
```html
<button on:click={() => $web3Modal.open()}>
  Connect Wallet
</button>
```

`package.json`
```json
{
	"name": "sveltekit-web3",
	"version": "1.0.0",
	"private": true,
	"scripts": {
		"generate": "prisma generate",
		"dev": "vite dev --port 5177",
		"build": "vite build",
		"preview": "vite preview",
		"check": "svelte-kit sync && svelte-check --tsconfig ./tsconfig.json",
		"check:watch": "svelte-kit sync && svelte-check --tsconfig ./tsconfig.json --watch",
		"lint": "prettier --check . && eslint .",
		"format": "prettier --write ."
	},
	"devDependencies": {
		"@playwright/test": "^1.28.1",
		"@sveltejs/adapter-auto": "^3.0.0",
		"@sveltejs/kit": "^2.5.7",
		"@sveltejs/vite-plugin-svelte": "^3.0.0",
		"@types/eslint": "^8.56.7",
		"autoprefixer": "^10.4.19",
		"eslint": "^9.0.0",
		"eslint-config-prettier": "^9.1.0",
		"eslint-plugin-svelte": "^2.36.0",
		"globals": "^15.0.0",
		"postcss": "^8.4.38",
		"prettier": "^3.1.1",
		"prettier-plugin-svelte": "^3.1.2",
		"svelte": "^4.2.7",
		"svelte-check": "^3.6.0",
		"tailwindcss": "^3.4.3",
		"tslib": "^2.4.1",
		"typescript": "^5.0.0",
		"typescript-eslint": "^7.5.0",
		"viem": "^2.12.4",
		"vite": "^5.2.8",
	},
	"type": "module",
	"peerDependencies": {
		"@wagmi/connectors": "^5.0.6",
		"@wagmi/core": "^2.0.0",
		"svelte": "^4.0.0 || ^5.0.0",
		"vite": "^4.0.0 || ^5.0.0"
	},
	"dependencies": {
		"@stablelib/random": "^1.0.2",
		"@web3modal/core": "^4.2.2",
		"@web3modal/ui": "^4.2.2",
		"@web3modal/wagmi": "^4.2.2",
		"siwe": "^1.1.6",
		"zod": "^3.21.4"
	}
}
```

`wagmi.ts`
```ts
import { writable, get } from 'svelte/store';
import {
	createConfig,
	http,
	getAccount,
	disconnect,
	watchAccount,
	reconnect,
	signMessage as wagmiSignMessage,
	type CreateConnectorFn,
	type GetAccountReturnType,
	type Config
} from '@wagmi/core';
import { mainnet, type Chain } from '@wagmi/core/chains';
import { createWeb3Modal, emailConnector, type Web3Modal } from '@web3modal/wagmi';
import { PUBLIC_WALLETCONNECT_ID } from '$env/static/public';
import { walletConnect, injected } from '@wagmi/connectors';

export const connected = writable<boolean>(false);
export const wagmiLoaded = writable<boolean>(false);
export const chainId = writable<number | null | undefined>(null);
export const signerAddress = writable<string | null>(null);
export const configuredConnectors = writable<CreateConnectorFn[]>([]);
export const loading = writable<boolean>(true);
export const web3Modal = writable<Web3Modal>();
export const wagmiConfig = writable<Config>();
export { mainnet };
import { randomStringForEntropy } from '@stablelib/random';
import { SiweMessage } from 'siwe';

type DefaultConfigProps = {
	connectors: CreateConnectorFn[]
	chains: Chain[],
	alchemyId?: string,
	walletConnectProjectId: string
};

const defaultOptions = {
	appName: 'My App',
	appDescription: 'This is my app.',
	appUrl: 'https://website.com',
	walletConnectProjectId: PUBLIC_WALLETCONNECT_ID,
	chains: [mainnet],
	connectors: [
		injected(),
		walletConnect({
			projectId: PUBLIC_WALLETCONNECT_ID
		})
	]
};

// Create config only
// Can be used in backend or frontend
export const generateConfig = (input: DefaultConfigProps) => {
	const connectors = [
		...input.connectors,
		emailConnector({
			options: {
				projectId: input.walletConnectProjectId
			}
		})
	];

	const transports = input.chains
		? input.chains.reduce(
				(acc, chain) => ({
					...acc,
					[chain.id]: http()
				}),
				{}
			)
		: {};

	const config = createConfig({
		chains: input.chains as [Chain, ...Chain[]],
		transports,
		connectors,
	});

	return config;
}

// Clientside only
export const init = async (input: {
	autoConnect?: boolean,
} = {}) => {
	// Create config
	const config = generateConfig(defaultOptions);

	// Setup stores 
	wagmiConfig.set(config);

	if (input?.autoConnect) reconnect(config);

	const modal = createWeb3Modal({
		wagmiConfig: config,
		projectId: PUBLIC_WALLETCONNECT_ID,
		enableAnalytics: true, // Optional - defaults to your Cloud configuration
		enableOnramp: true // Optional - false as default
	});

	web3Modal.set(modal);
	wagmiLoaded.set(true);

	try {
		// Setup listeners
		watchAccount(get(wagmiConfig), {
			onChange(data) {
				handleAccountChange(data);
			}
		});

		// Init
		const account = await waitForConnection();
		if (account.address) {
			const chain = get(wagmiConfig).chains.find((chain) => chain.id === account.chainId);
			if (chain) chainId.set(chain.id);
			connected.set(true);
			signerAddress.set(account.address);
		}
		loading.set(false);
	} catch (err) {
		loading.set(false);
	}
};

const handleAccountChange = (data: GetAccountReturnType) => {
	// Wrap the original async logic in an immediately invoked function expression (IIFE)
	return (async () => {
		if (get(wagmiLoaded) && data.address) {
			const chain = get(wagmiConfig).chains.find((chain) => chain.id === data.chainId);

			if (chain) chainId.set(chain.id);
			connected.set(true);
			loading.set(false);
			signerAddress.set(data.address);
		} else if (data.isDisconnected && get(connected)) {
			loading.set(false);
			await disconnectWagmi(); // Handle async operation inside
		}
	})();
};

export const disconnectWagmi = async () => {
	await disconnect(get(wagmiConfig));
	connected.set(false);
	chainId.set(null);
	signerAddress.set(null);
	loading.set(false);
};

const waitForConnection = (): Promise<GetAccountReturnType> =>
	new Promise((resolve, reject) => {
		const attemptToGetAccount = () => {
			const account = getAccount(get(wagmiConfig));
			if (account.isDisconnected) reject('account is disconnected');
			if (account.isConnecting) {
				setTimeout(attemptToGetAccount, 250);
			} else {
				resolve(account);
			}
		};

		attemptToGetAccount();
	});


export const generateMessage = async (input: {
	address: string,
	origin: string,
	host: string
}) => {
	const message = new SiweMessage({
		domain: input.host,
		address: input.address,
		statement: 'Sign in to Cyfrin',
		uri: input.origin,
		version: '1',
		chainId: mainnet.id,
		nonce: randomStringForEntropy(96)
	});

	return message;
};

export const signMessage = async (input: {
	message: string
}) => {
	const config = generateConfig(defaultOptions);

	const signature = await wagmiSignMessage(
		config,
		{
			message: input.message
		}
	);

	return signature;
};

export const validateMessage = async (input: {
	signature: string,
}) => {
	const siwe = new SiweMessage(JSON.parse(input.signature) || '{}');

	const result = await siwe.validate();
	
	return {
		result,
		siwe
	}
};

// Things not used in profiles app
// but were in the original svelte-wagmi code
// export const WC = async () => {
// 	try {
// 		get(web3Modal).open();
// 		await waitForAccount();

// 		return { succcess: true };
// 	} catch (err) {
// 		return { success: false };
// 	}
// };

// const waitForAccount = () => {
// 	return new Promise((resolve, reject) => {
// 		const unsub1 = get(web3Modal).subscribeEvents((newState) => {
// 			if (newState.data.event === 'MODAL_CLOSE') {
// 				reject('modal closed');
// 				unsub1();
// 			}
// 		});
// 		const unsub = watchAccount(get(wagmiConfig), {
// 			onChange(data) {
// 				if (data?.isConnected) {
// 					// Gottem, resolve the promise w/user's selected & connected Acc.
// 					resolve(data);
// 					unsub();
// 				} else {
// 					console.warn('🔃 - No Account Connected Yet...');
// 				}
// 			}
// 		});
// 	});
// };

// type DefaultConfigProps = {
// 	appName: string;
// 	appIcon?: string | null;
// 	appDescription?: string | null;
// 	appUrl?: string | null;
// 	autoConnect?: boolean;
// 	alchemyId?: string | null;
// 	chains?: Chain[] | null;
// 	connectors: CreateConnectorFn[];
// 	walletConnectProjectId: string;
// };
```
