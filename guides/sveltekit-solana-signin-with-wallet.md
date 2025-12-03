
# Sveltekit + Solana Wallet Signin

Full-stack SvelteKit example project showing "Sign in With Solana" patterns. - Use Solana wallet adapter & connect to wallet. - Request time-sensitive message from the backend. - Sign message using wallet. - Verify signature.

*This uses an older version of SvelteKit (2022) but the overall patterns should still be useful.*
![](https://raw.githubusercontent.com/qudo-code/public/refs/heads/main/media/Pasted%20image%2020251202200621.png)
![](https://raw.githubusercontent.com/qudo-code/public/refs/heads/main/media/Pasted%20image%2020251202200645.png)
It all starts in +layout.svelte, the file that is the wrapper for all pages. This handles some initial setup and gets base components and providers on the page.
![](https://raw.githubusercontent.com/qudo-code/public/refs/heads/main/media/Pasted%20image%2020251202200659.png)
Once the page is set up, this +page.svelte is rendered. This page waits for a wallet connection, and upon "connected", reaches out to our endpoint to get a time-sensitive message, sign the message, and verify the signature we just generated.
![](https://raw.githubusercontent.com/qudo-code/public/refs/heads/main/media/Pasted%20image%2020251202200710.png)
Here is the endpoint /api/solana/message/verify that is responsible for verifying the TOTP and the signature generated and sent from UI.
![](https://raw.githubusercontent.com/qudo-code/public/refs/heads/main/media/Pasted%20image%2020251202200726.png)

See repo for additional examples around verifying nonce hasn't already been used. 
