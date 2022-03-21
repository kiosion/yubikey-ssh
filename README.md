<div align=center>
  <h1>How to use a Yubikey for SSH authentication</h1>
</div>

I decided to create this short guide as I couldn't find much in the way of guides when setting this up myself, and initially found the process a bit confusing myself (I'm by no means an expert with this stuff).

All the guides I *did* find on using a Yubikey for SSH authentication involved using its Smart Card slot for GPG, which is unnecessary and over-complicated as using [yubikey-agent](https://github.com/FiloSottile/yubikey-agent), we can use the Authentication slot (9a), and leave the Smart Card slot open for other uses.

## SETUP

### 1. Install yubikey-manager

#### Linux
yubikey-manager can be used as either an AppImage from [Yubikey's website](https://www.yubico.com/support/download/yubikey-manager/) or installed from your package manager. On Arch Linux, it's avaliable under the name [`yubikey-manager-qt`](https://archlinux.org/packages/community/x86_64/yubikey-manager-qt/); or [`yubikey-manager`](https://archlinux.org/packages/community/x86_64/yubikey-manager/) for the CLI version.

After installation, `pcscd.service` needs to be enabled and started: `sudo systemctl enable pcscd.service --now`. 

yubikey-manager can be found under the name `ykman-gui`;  or `ykman` for the CLI version.

#### MacOS

yubikey-manager can be downloaded and installed from [Yubikey's website](https://www.yubico.com/support/download/yubikey-manager/).

### 2. Install yubikey-agent

#### Linux
yubikey-agent can be installed using the [`yubikey-agent`](https://aur.archlinux.org/packages/yubikey-agent/) package from the AUR, or on other distros, can be [installed manually](https://github.com/FiloSottile/yubikey-agent/blob/main/systemd.md).

After installation, it needs to be enabled and started: `systemctl daemon-reload --user && systemctl --user enable yubikey-agent.service --now`.

#### MacOS
yubikey-agent can be installed using Homebrew using `brew install yubikey-agent`. After installation, it needs to be enabled using `brew services start yubikey-agent`.

## STEPS

### 1. Generate a new key
Here, you could either choose `yubikey-agent -setup`, or, use yubikey-manager to generate a key. I chose the second option, as although yubikey-agent is easier, it didn't give me the granularity I wanted. 

It's also important to note that `yubikey-agent -setup` generates a random Management Key and stores it in PIN-protected metadata[^1], making it difficult to use the other PIV slots.

#### a. Using yubikey-manager
To generate a new key using the yubikey-manager software, navigate to the 'Applications > PIV' tab, and make sure the 'Authentication' slot is selected. If there's already a certificate present, **make sure** it's not important and not in use. Overwriting the slot is **not** reversible.

Click 'Generate', select the key type you want to generate (the default should be fine if you're not sure), name it 'SSH Key', and set the expiration date.

If everything looks good, save it and enter the PIN to confirm. If you haven't set one previously, the default is `123456` - This should be changed to something more secure, along with the PUK and Management Key. Save these somewhere offline and safe.

#### b. Using yubikey-agent
To generate a new key with yubikey-agent and store it in your Yubikey's PIV authentication slot, simply run `yubikey-agent -setup` with your yubikey connected and follow the steps.

You'll be asked to set a PIN, and be given your public key once the process is finished.

### 2. Configure your shell
Next, you'll need to configure your shell to set the `SSH_AUTH_SOCK` environment variable on launch.

First find the location of the yubikey-agent socket. You can look at the output of `ps aux | grep yubikey-agent` if you're not sure. For me on Linux, its location was `/run/user/{Your UID}/yubikey-agent/yubikey-agent.sock` by default.

#### Fish
Open your config and add  `set -gx SSH_AUTH_SOCK "/run/user/{Your UID}/yubikey-agent/yubikey-agent.sock"`, where the path is the location of the yubikey-agent socket.

#### Bash / ZSH
Open your config and add `export SSH_AUTH_SOCK="/run/user/{Your UID}/yubikey-agent/yubikey-agent.sock"`, where the path is the location of the yubikey-agent socket.

## NOTES

That's it! You should now have a fully configured and working SSH key in your Yubikey's Authentication slot. 

- To print the public key and test the setup, run `ssh-add -L`. 
	- If you don't see any identities, or get an error, double-check all the required services are installed and running, and that you've configured your shell's SSH_AUTH_SOCK environment variable correctly.
- If not using the '-setup' flag with yubikey-agent, *make sure* you set a secure PIN, PUK, and Management Key and store them somewhere safe and offline.
- If you run into issues using the Yubikey for other functionality after using it with yubikey-agent, I've found either unplugging-and-replugging it, or sending a SIGHUP to the yubikey-agent process solves this. 

[^1]: See yubikey-agent's [manual setup instructions](https://github.com/FiloSottile/yubikey-agent#manual-setup-and-technical-details)
