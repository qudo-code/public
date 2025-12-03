
# Computer Setup
![](https://raw.githubusercontent.com/qudo-code/public/refs/heads/main/media/Pasted%20image%2020251202164036.png)
#### MacOS w/Aerospace
![](https://raw.githubusercontent.com/qudo-code/public/refs/heads/main/media/Pasted%20image%2020251202164159.png)
#### NixOS w/Hyprland
![](https://raw.githubusercontent.com/qudo-code/public/refs/heads/main/media/Pasted%20image%2020251202164227.png)
## Setup `dots` CLI

I've added a config pattern and CLI for detecting the current OS and syncing configs based on the environment.

Files to sync/copy are tracked in `~/dots/config.sh`.
#### Install

Clone and install the `dots` CLI with the following.

```bash
git clone git@github.com:qudo-code/dots.git ~/dots && ~/dots/install.sh
```

#### Usage
Once installed, the `dots` CLI should be available in your terminal.
- `dots sync`: Pushes any changes GitHub, Detects OS, copies files from config to system.
   - Change `~/dots/config.sh` as needed.
- `dots edit`: Opens the config files in editor.
	- First checks for `zed`, then `cursor`, then `code`.
- `dots install`: Intended to be used for heavier less frequest setup tasks.
	- On `nixos`: Rebuilds the current config.
	- On `macos`: Installs Homebrew, apps via homebrew and other base apps and tools.
*CLI logic lives in ~/dots/cli.sh, edit as needed.*

```
Dots CLI
OS:      macos
Home:    /Users/qudo
Command: Help
-----------------
✏️ Open configs in editor
~/dots edit
~/dots e

➡️ Sync configs with sytem
~/dots sync
~/dots s

🛠️ Run ~/dots install step
~/dots install
~/dots i
```

[View Repo](https://github.com/qudo-code/dots)
