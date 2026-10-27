# 🚀 OmniRush AI on Android with Termux

Run the **OmniRush AI coding agent on an Android phone** using **Termux + Debian (proot)** — no PC required.

> ✅ This setup was tested successfully on an **ARM64/aarch64 Android phone** with OmniRush CLI **2.7.3** and Linux ARM64 Node.js **22.23.4**.

## What this gives you

After setup you can use OmniRush from your Android terminal to:

- chat with coding models
- read and edit project files
- run commands and tests
- work inside Git repositories
- use the familiar OpenCode-style terminal UI
- launch it from Termux with a simple `omni` command

OmniRush currently provides free model access with account usage limits. **Free access, models, and token allowances can change**, so check your OmniRush Usage page for your own current limits.

## Models

The tested account showed models including:

- GPT 6 Astra
- GPT 6 Sol
- GPT 6.1 Sol
- Claude Opus 5.5

Model availability can change over time.

---

# Requirements

You need:

- an **ARM64/aarch64 Android phone**
- [Termux](https://termux.dev/) installed
- an internet connection
- an OmniRush account

You do **not** need root for this setup.

Check your phone architecture in Termux:

```bash
uname -m
```

Expected:

```text
aarch64
```

---

# Step 1 — Update Termux and install proot-distro

Open normal Termux and run:

```bash
pkg update
pkg install proot-distro -y
```

---

# Step 2 — Install Debian

```bash
proot-distro install debian
```

When it finishes, enter Debian:

```bash
proot-distro login debian
```

Your prompt should change to something similar to:

```text
root@localhost:~#
```

Check the architecture again:

```bash
uname -m
```

Expected:

```text
aarch64
```

---

# Step 3 — Install basic Debian tools

Inside Debian run:

```bash
apt update && apt install -y curl ca-certificates gnupg git
```

---

# Step 4 — Install NVM

Inside Debian:

```bash
/usr/bin/curl -fsSL https://raw.githubusercontent.com/nvm-sh/nvm/master/install.sh | bash
```

Load NVM immediately:

```bash
export NVM_DIR="$HOME/.nvm"
. "$NVM_DIR/nvm.sh"
```

---

# Step 5 — Install Linux ARM64 Node.js 22

```bash
nvm install 22
```

Verify that this is the **Linux** build, not Termux's Android build:

```bash
node -p "process.platform + ' ' + process.arch"
node --version
```

Expected platform:

```text
linux arm64
```

A working tested version was:

```text
v22.23.4
```

> ⚠️ Important: if the first command says `android arm64`, you are still using Termux's Android Node binary. OmniRush should use the Linux ARM64 Node installed by NVM inside Debian.

---

# Step 6 — Install OmniRush

Still inside Debian:

```bash
npm i -g omnirush@latest
```

Check it:

```bash
omnirush --version
```

Tested output:

```text
omnirush 2.7.3
```

A newer version is fine.

---

# Step 7 — Sign in

Run:

```bash
omnirush login
```

OmniRush will display a device code and a web link.

Open the link in your Android browser, approve the login, then return to Termux.

Verify your account:

```bash
omnirush whoami
```

If the response shows your account as active and `model_access` is `true`, login worked.

---

# Step 8 — Test OmniRush

Create a small test project:

```bash
mkdir -p ~/omnirush-test
cd ~/omnirush-test
git init
omnirush
```

Choose a model such as **GPT 6 Astra** and ask:

```text
Create a file named hello.txt with the text: OmniRush works on Android.
```

Exit OmniRush and check:

```bash
cat hello.txt
```

Expected:

```text
OmniRush works on Android.
```

If you see that, OmniRush can successfully edit files on your Android setup. 🎉

---

# Step 9 — Create a one-word `omni` launcher

Normally you would have to enter Debian, load NVM, open the project and launch OmniRush manually.

This shortcut lets you simply do:

```bash
cd ~/my-project
omni
```

First leave Debian so you are back in normal Termux:

```bash
exit
```

Then paste this in **normal Termux**:

```bash
cat > "$PREFIX/bin/omni" <<'EOF'
#!/data/data/com.termux/files/usr/bin/bash

PROJECT="$PWD"

proot-distro login debian \
  --bind "$PROJECT:/workspace" \
  -- bash -lc '
    export NVM_DIR="/root/.nvm"
    . "$NVM_DIR/nvm.sh"
    cd /workspace
    exec omnirush
  '
EOF

chmod +x "$PREFIX/bin/omni"
```

Now you can launch OmniRush from any Termux project folder:

```bash
cd ~/my-project
omni
```

The current folder is mounted inside Debian as `/workspace`, so OmniRush works on that project.

---

# ⚠️ About the sandbox warning

On Android with Debian/proot, OmniRush may show:

```text
The sandbox can't run on this computer: no sandbox backend works here
(install bubblewrap, Podman or Docker). This chat runs on this machine.
```

This is **expected** with this Android setup.

OmniRush still works, but commands are executed directly in the Debian/proot environment rather than inside OmniRush's normal extra sandbox layer.

### What this means

Be more careful with agent permissions:

- work inside a project folder
- review destructive commands before allowing them
- keep backups / Git commits
- do not give the agent passwords, API keys, recovery codes or other secrets
- avoid running OmniRush as Android root

For ordinary coding work, file editing and tests can still work without the sandbox.

---

# Daily use

Once everything is installed, the normal workflow is only:

```bash
cd ~/your-project
omni
```

That's it. 🚀

---

# Useful commands

### Check OmniRush version

```bash
omnirush --version
```

### Check account and usage

Inside Debian:

```bash
omnirush whoami
```

### Enter Debian manually

```bash
proot-distro login debian
```

### Load NVM manually inside Debian

```bash
export NVM_DIR="$HOME/.nvm"
. "$NVM_DIR/nvm.sh"
```

### Update OmniRush

```bash
omnirush update
```

---

# Troubleshooting

## `omnirush: command not found`

Inside Debian, load NVM:

```bash
export NVM_DIR="$HOME/.nvm"
. "$NVM_DIR/nvm.sh"
```

Then check:

```bash
which node
which omnirush
```

## Node reports `android arm64`

You are using Termux's Node instead of Debian's Linux Node.

Inside Debian run:

```bash
export NVM_DIR="$HOME/.nvm"
. "$NVM_DIR/nvm.sh"
nvm use 22
```

Then verify:

```bash
node -p "process.platform + ' ' + process.arch"
```

It should say:

```text
linux arm64
```

## Debian is not installed

In normal Termux:

```bash
proot-distro install debian
```

## Login browser does not open

`omnirush login` prints a URL. Copy that URL and open it manually in your Android browser.

## Sandbox warning

See the [sandbox section](#️-about-the-sandbox-warning). It is expected under Android/proot.

---

# How it works — simple explanation

Termux itself runs Android binaries. OmniRush expects a normal Linux environment.

So this setup creates a small Debian Linux environment inside Android using `proot-distro`. NVM then installs a real **Linux ARM64 Node.js** inside Debian, and OmniRush runs on top of that.

```text
Android
  ↓
Termux
  ↓
Debian (proot)
  ↓
Linux ARM64 Node.js
  ↓
OmniRush AI
```

The `omni` shortcut hides those layers during normal use.

---

# Notes about “free” access

This repository does **not** promise unlimited AI usage.

OmniRush currently offers free access with account-specific token allowances and eligibility rules. Your allowance may depend on your account, linked services, promotions, quality programs or future policy changes.

Always check the official OmniRush website and your **Usage** page for the current numbers.

- Official site: https://omnirush.ai/
- npm package: https://www.npmjs.com/package/omnirush

---

## Tested successfully ✅

The full flow in this guide was tested on a real ARM64 Android device:

```text
Termux → Debian proot → Linux ARM64 Node 22 → OmniRush → login → model selection → file editing
```

No PC was required.

---

If this guide helped you, consider ⭐ starring the repo so other Android users can find it.