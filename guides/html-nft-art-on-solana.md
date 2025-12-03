# HTML NFT Art on Solana
![](https://raw.githubusercontent.com/qudo-code/public/refs/heads/main/media/Pasted%20image%2020251203133842.png)
I figured out how to make an interactive HTML NFT game that's playable on [@exchgART](https://x.com/exchgART). Here's what I learned and how you can build one too!

The first place I started was inspecting the metadata of the only other interactive NFT that I knew about at the time, degenpoets "Chicago Hacker House". An HTML book of poems by Degen Poet X [@kidonthephoenix](https://x.com/kidonthephoenix). Super cool, check it out.
https://exchange.art/single/Eum7LnRRrDoenfMrU91V93URtHGSfwbXnC8pfUnBKEe

When looking at the metadata, couple of things stood out. There was a file with type: text/html as well as an image file. This HTML file was uploaded to Arweave and also referenced in the animation_url property.
![](https://raw.githubusercontent.com/qudo-code/public/refs/heads/main/media/Pasted%20image%2020251203134146.png)
Metadata URL: https://yypd57m7s5dkgimdzqsywbtgzegg6yemx3ka6v5sq24i7g5aqtza.arweave.net/xh4-_Z-XRqMhg8wliwZmyQxvYIy-1A9Xsoa4j5ughPI

So I thought, how hard can it be to do it from scratch? I made a new repo, I got an arweave wallet, I got my app hosted on arweave. But then, out of all places to get stuck, I couldn't get the .gif cover image uploaded correctly...So, that was fun, but let's go use UIs.

![](https://raw.githubusercontent.com/qudo-code/public/refs/heads/main/media/Pasted%20image%2020251203134319.png)
![](https://raw.githubusercontent.com/qudo-code/public/refs/heads/main/media/Pasted%20image%2020251203134325.png)
![](https://raw.githubusercontent.com/qudo-code/public/refs/heads/main/media/Pasted%20image%2020251203134333.png)

If we go to [http://Exchange.art](https://t.co/E5EqK9RVJP), they give us the ability to upload an HTML file. They also let you provide a cover image which acts as the fallback image for wallets and platforms that can't render the HTML version. I chose to use a .gif of gameplay as my cover image.

![](https://raw.githubusercontent.com/qudo-code/public/refs/heads/main/media/Pasted%20image%2020251203134347.png)

To make my life easier (and hopefully yours), I've developed a Svelte template repository aimed at building HTML NFTs. This build system is set up to build all of your HTML/SCSS/CSS/TS/JS to a single HTML file ready for [http://Exchange.art](https://t.co/E5EqK9RVJP) upload.

Template: https://github.com/qudo-code/template--svelte-single-file

If a framework is more than your project requires, check out this Medium article that I wrote which goes over a simple HTML & CSS demo project.
https://medium.com/@qudo_40051/creating-an-interactive-html-nft-on-solana-46ab19d116e0
