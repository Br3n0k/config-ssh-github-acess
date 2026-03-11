Claro. Vou te entregar já em formato pronto para criar o repositório.

## Descrição do repositório

**A practical step-by-step guide to configuring SSH access to GitHub on Linux servers and local machines, with examples for Alpine Linux and general Git setup.**

---

## README.md completo

````markdown
# GitHub SSH Setup Guide

A practical, beginner-friendly tutorial for configuring SSH access to GitHub on Linux servers and local machines.

This guide focuses on a clean and secure setup using SSH keys instead of passwords, with examples that work especially well on Alpine Linux, but also apply to most Linux distributions.

---

## Why use SSH with GitHub?

Using SSH is the recommended way to authenticate Git operations with GitHub because it is:

- More secure than using username/password
- More convenient for servers and automation
- Better suited for private repositories
- Ideal for development, deployment, and CI/CD workflows

GitHub no longer supports password authentication for Git over HTTPS, so SSH is often the best long-term solution.

---

## What you will learn

In this tutorial, you will learn how to:

- Install and verify SSH tools
- Generate an SSH key pair
- Add the public key to your GitHub account
- Test the connection to GitHub
- Clone repositories using SSH
- Configure Git identity
- Optionally use a custom SSH config file

---

## Requirements

Before starting, make sure you have:

- A GitHub account
- Access to a Linux machine or server
- Git installed
- Shell access to the machine

For Alpine Linux, this tutorial assumes a minimal server environment.

---

## 1. Install Git and OpenSSH

### Alpine Linux

```bash
apk update
apk add git openssh
````

### Debian / Ubuntu

```bash
sudo apt update
sudo apt install git openssh-client -y
```

### Verify installation

```bash
git --version
ssh -V
```

---

## 2. Generate a new SSH key

The recommended key type is **ed25519**.

Run:

```bash
ssh-keygen -t ed25519 -C "your-email@example.com"
```

You will see prompts like:

```text
Enter file in which to save the key (/root/.ssh/id_ed25519):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
```

### Recommended behavior

* Press `Enter` to use the default file location
* On servers, you may leave the passphrase empty for unattended access
* On personal machines, using a passphrase is recommended

This will create two files:

```text
~/.ssh/id_ed25519
~/.ssh/id_ed25519.pub
```

Where:

* `id_ed25519` is your private key
* `id_ed25519.pub` is your public key

---

## 3. View and copy the public key

Run:

```bash
cat ~/.ssh/id_ed25519.pub
```

You will get output similar to:

```text
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIBExampleKeyHere your-email@example.com
```

Copy the entire line exactly as shown.

---

## 4. Add the public key to GitHub

Open GitHub in your browser and go to:

* **Settings**
* **SSH and GPG keys**
* **New SSH key**

Then:

* Give the key a descriptive title, such as `Alpine Server` or `Production Node 01`
* Paste your public key
* Save it

### Important

If you want to access repositories from an organization, your GitHub account must already have permission to those repositories.
The SSH key authenticates **you**, not the organization itself.

---

## 5. Test SSH authentication with GitHub

Run:

```bash
ssh -T git@github.com
```

The first time, you may see:

```text
The authenticity of host 'github.com (IP_ADDRESS)' can't be established.
ED25519 key fingerprint is ...
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```

Type:

```text
yes
```

If everything is correct, GitHub should respond with something similar to:

```text
Hi username! You've successfully authenticated, but GitHub does not provide shell access.
```

That message confirms your SSH access is working.

---

## 6. Clone repositories using SSH

To clone a repository, use the SSH URL instead of HTTPS.

### Example

```bash
git clone git@github.com:OWNER/REPOSITORY.git
```

For example:

```bash
git clone git@github.com:my-org/my-private-repo.git
```

If you use the HTTPS URL, Git may ask for credentials or tokens instead of using your SSH key.

---

## 7. Configure Git identity

This step is separate from SSH authentication.

Set your Git username and email:

```bash
git config --global user.name "Your Name"
git config --global user.email "your-email@example.com"
```

Verify:

```bash
git config --global --list
```

Example output:

```text
user.name=Your Name
user.email=your-email@example.com
```

This information will be used in your commits.

---

## 8. Optional: create an SSH config file

If you want more control over how SSH behaves, create this file:

```bash
nano ~/.ssh/config
```

Add:

```ssh
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519
```

Save and exit.

Then set correct permissions:

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/config
chmod 600 ~/.ssh/id_ed25519
chmod 644 ~/.ssh/id_ed25519.pub
```

This is especially useful when you manage multiple keys.

---

## 9. Optional: use a custom key name

If you want to create a key with a custom name:

```bash
ssh-keygen -t ed25519 -C "your-email@example.com" -f ~/.ssh/github_alpine
```

Then your files will be:

```text
~/.ssh/github_alpine
~/.ssh/github_alpine.pub
```

And your SSH config should be:

```ssh
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/github_alpine
```

---

## 10. Recommended permissions for SSH files

SSH is strict about file permissions.

Run:

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_ed25519
chmod 644 ~/.ssh/id_ed25519.pub
```

If permissions are too open, SSH may refuse to use the key.

---

## 11. Troubleshooting

### Permission denied (publickey)

If you see:

```text
Permission denied (publickey)
```

Check the following:

* Your public key was added to the correct GitHub account
* You are cloning with the SSH URL, not HTTPS
* Your private key exists in `~/.ssh/`
* File permissions are correct
* Your SSH config points to the right key
* Your GitHub account has access to the target repository

You can debug with:

```bash
ssh -vT git@github.com
```

---

### Repository not found

If SSH works but cloning fails with:

```text
Repository not found.
```

This usually means:

* The repository URL is incorrect
* Your GitHub account does not have permission to access the repository
* You are trying to access an organization repository without being granted access

---

### Wrong remote URL

Check the current remote:

```bash
git remote -v
```

If needed, change it:

```bash
git remote set-url origin git@github.com:OWNER/REPOSITORY.git
```

---

## 12. Security best practices

For production and server environments, consider the following:

* Use one SSH key per server
* Name keys clearly
* Revoke unused keys from GitHub
* Avoid reusing personal workstation keys on production servers
* Prefer deploy keys for single-repository server access
* Prefer machine users or fine-scoped access for automation when needed

---

## 13. Deploy keys vs personal SSH keys

There are two common ways to give a server access to GitHub:

### Personal SSH key

Best when:

* The server acts on behalf of your GitHub user
* You need access to multiple repositories you already have access to

### Deploy key

Best when:

* A single server only needs access to one repository
* You want tighter security boundaries
* You want repository-specific access

Deploy keys are often the better option for production servers.

---

## 14. Example full flow on Alpine Linux

```bash
apk update
apk add git openssh

ssh-keygen -t ed25519 -C "your-email@example.com"

cat ~/.ssh/id_ed25519.pub

ssh -T git@github.com

git config --global user.name "Your Name"
git config --global user.email "your-email@example.com"

git clone git@github.com:OWNER/REPOSITORY.git
```

---

## 15. Common commands reference

### Show public key

```bash
cat ~/.ssh/id_ed25519.pub
```

### Test GitHub SSH connection

```bash
ssh -T git@github.com
```

### Clone repository with SSH

```bash
git clone git@github.com:OWNER/REPOSITORY.git
```

### Show Git configuration

```bash
git config --global --list
```

### Show current remotes

```bash
git remote -v
```

### Change remote to SSH

```bash
git remote set-url origin git@github.com:OWNER/REPOSITORY.git
```

---

## Conclusion

You now have a secure and practical SSH setup for GitHub.

This method works well for:

* Alpine Linux servers
* Development machines
* Private repositories
* Organization repositories
* Deployment and automation workflows

If you found this repository useful, feel free to fork it or adapt it for your own infrastructure documentation.

---

## License

This project is open for educational use.
You may adapt and reuse this tutorial in your own projects.
