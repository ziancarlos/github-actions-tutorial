# SSH Key Pair

* A way to access a VPS through SSH **without entering a password every time**.

## Create an SSH Key Pair

```bash
ssh-keygen -t ed25519 -C "github-actions-vps"
```

```text
ssh-keygen -t ed25519 -C "github-actions-vps"
              ↑                  ↑
        key algorithm       comment / label
```

This generates an **SSH key pair**:

```text
                 SSH KEY PAIR
               ┌───────────────┐
               │               │
        🔑 PRIVATE         🔓 PUBLIC
           KEY                KEY
               │               │
               ↓               ↓
       Keep it secret!    Register it
                           on the VPS
```

The private key and public key are mathematically related.

The key pair is **not directly tied to the computer that generated it**. The computer is simply where the key pair was generated and stored.

You can copy the private key to another machine, but you should **never share the private key**.

## Register the Public Key on the VPS

First, display the public key:

```bash
cat ~/.ssh/id_ed25519.pub
```

It will return something like:

```text
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAA... github-actions-vps
```

SSH into your VPS:

```bash
ssh vpsname@YOUR_VPS_IP
```

Create the `.ssh` directory:

```bash
mkdir -p ~/.ssh
```

Set the correct permissions:

```bash
chmod 700 ~/.ssh
```

Open the `authorized_keys` file:

```bash
nano ~/.ssh/authorized_keys
```

Paste your **public key** into this file:

```text
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAA... github-actions-vps
```

Save the file.

You can also make sure the file has appropriate permissions:

```bash
chmod 600 ~/.ssh/authorized_keys
```

## Access the VPS Using the Private Key

From the machine that has the private key:

```bash
ssh -i /path/to/private/key vpsname@YOUR_VPS_IP
```

For example:

```bash
ssh -i ~/.ssh/id_ed25519 vpsname@YOUR_VPS_IP
```

The process is:

```text
                 YOUR MACHINE
                      │
              🔑 PRIVATE KEY
                      │
                      │ SSH
                      ↓
                    VPS
                      │
              ~/.ssh/authorized_keys
                      │
              🔓 PUBLIC KEY
```

The VPS does **not** receive your private key.

It only has your **public key** in:

```text
~/.ssh/authorized_keys
```

When you connect, SSH uses the private key to prove that you are allowed to use the corresponding public key.

### Important

```text
PRIVATE KEY
❌ Never share
❌ Never put in GitHub repository
❌ Never put in authorized_keys

PUBLIC KEY
✅ Safe to register on the VPS
✅ Goes inside ~/.ssh/authorized_keys
```

For GitHub Actions, the usual setup is:

```text
GitHub Actions
      │
      │ 🔑 private key
      │
      ↓
     VPS
      │
      │ 🔓 matching public key
      │
~/.ssh/authorized_keys
```

The GitHub Actions secret stores the **private key**, while the VPS stores the corresponding **public key**.
