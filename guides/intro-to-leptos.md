
# Intro to Leptos
https://x.com/_qudo/status/1638193893580869632
gm, this week I've been learning Rust. As I've been learning this from a web devs perspective, I've been eager to find cool web use cases for Rust. I looked at a couple Rust/WASM frameworks and this one stood out. Here's what I've learned so far...
![](https://raw.githubusercontent.com/qudo-code/public/refs/heads/main/media/Pasted%20image%2020251202194805.png)
I am new to this framework and Rust so DYOR, what do I know. But, I've experimented with a lot of state shapes in web dev using meta frameworks like SvelteKit and lately the closest thing to what I've wanted is tRPC + Tanstack Query + tRPC/Tanstack Adapter.

This has been a cool way to componentize server interactions. You get to break out of file based routing for a few things (whew), have nice subscribables as Svelte stores. It's nice, esp with Prisma integration and stuff it ALMOST feels like one piece, kinda..

I appreciate all these tools, and what Svelte does with the compiler and no vdom and fine tuned reactivity. But it seems like instead of layering on so many pieces just to get a nice DX, we could use a new language to compose web apps as a whole.

The above is something I've always had in the back of my mind. Recently, I've been really involved in Solana things and I started learning Rust because that's the popular choice for writing programs. At some point I thought, "I wonder if you can make UI with this stuff"?

I cam across Leptos and it seems pretty sweet. Most of the other Rust to web app frameworks seemed Reactlike....so

Here's an example Leptos component that contains some methods, and a template that references some reactive state.
![](https://raw.githubusercontent.com/qudo-code/public/refs/heads/main/media/Pasted%20image%2020251202194852.png)

If you want server logic, that looks like this. Here's an endpoint that toggles a cookie value.
![](https://raw.githubusercontent.com/qudo-code/public/refs/heads/main/media/Pasted%20image%2020251202194901.png)
Both of these things can be in the same file. A file can have some server bits plus some UI bits. If a "view" or template calls a server fn on the client side, it will end up being a request to the backend to run that fn.

This seems like a cool benefit of generating both the client and the server from a low level. There isn't really frontend and backend, it's all just data flows and, "app".

![](https://raw.githubusercontent.com/qudo-code/public/refs/heads/main/media/Pasted%20image%2020251202194926.png)
Here's another version of the pervious logic that runs on the server during initial generation.
![](https://raw.githubusercontent.com/qudo-code/public/refs/heads/main/media/Pasted%20image%2020251202194941.png)
Here's the component and view for the dark mode logic. Leptos also provides cool form actions for managing/submitting form state. Another cool thing about these actions is that they will work and be available on the page even before WASM hydration.
![](https://raw.githubusercontent.com/qudo-code/public/refs/heads/main/media/Pasted%20image%2020251202194952.png)
This was an awesome intro to the Leptos framework with the creator,
[@greg_johnston](https://x.com/greg_johnston), and it goes over this dark mode example in more detail.
https://www.youtube.com/watch?v=AD3FHodVgE8&embeds_referring_euri=https%3A%2F%2Fx.com%2F&source_ve_path=MjM4NTE

If you want to learn even more, here's the repo. I can't wait to start building some apps with this. It seems like a cool engineering solution to my problems and a great excuse to continue learning Rust. LMK if anything above needs revision.
https://github.com/leptos-rs
