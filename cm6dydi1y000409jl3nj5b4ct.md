---
title: "How to Manage Multiple Git Accounts on One Device"
seoTitle: "Guide to Managing Multiple Git Accounts with SSH"
seoDescription: "Learn how to manage multiple Git accounts on one device using SSH keys and gitconfig. A step-by-step guide to streamline personal and work projects."
datePublished: Sun Jan 26 2025 18:28:21 GMT+0000 (Coordinated Universal Time)
cuid: cm6dydi1y000409jl3nj5b4ct
slug: how-to-manage-multiple-git-accounts-on-one-device
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/842ofHC6MaI/upload/064d2e424d1b96548b16e53460bf5b82.jpeg
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1737915948772/6fade393-c8c4-4870-93b7-33dfdfc253f0.jpeg

---

Ever wondered having multiple `git` accounts in a same device, say if you’re working on a personal project and on another work project, you’ve configured two git accounts, but accidentally you push to via your personal account where you should have pushed with your work account. Yep, accidents do happen you can revert those changes, but learning from your mistakes should is a necessity.

So here I am, with this quick guide on configuring more than one `git` accounts. We’ll be using `SSH` keys and some basic `gitconfig` syntax.

Disclaimer: It is advised to read this blog in **dark mode**, to enhance the overall experience.

## SSH vs HTTPS 🤔

We know in GitHub, we usually get two options to `clone` a repository(I’m ignoring `github-cli` here) .  
And, Most Version Control Software support authentication with SSH or HTTPS: the preferred way is **always using SSH where possible**.  
Git does not cache the user's credentials by default, so with HTTPS, you need to re-enter your `PAT` (Personal Access Token) each time you perform a clone, push, or pull.  
On the other hand, SSH is safer and, when set up correctly (which will be once you follow this guide), you can completely forget about it!

%[https://media.giphy.com/media/krqnIPLK0EAduikysz/giphy.gif?cid=ecf05e47ac5t0gqjiz501nxmgmhvesgim495a9lpy3yeahj2&ep=v1_gifs_search&rid=giphy.gif&ct=g] 

## Generating SSH keys

Before generating the keys, you need to know this equation,`One SSH key pair = One git config`.

We’ll be using `rsa` algorithm, for generating our SSH key pairs.

```bash
ssh-keygen -t rsa -b 4096 -C "your_personal_email@example.com" -f ~/.ssh/<personal_key>
```

\-t : This flag is used for specifying the type of the key to be used. For example, rsa, ed25519, etc..

\-C: For providing a comment, here, we’re using a meaningful comment for identifying keys by providing our emails

\-f : To specify the filename of the key pair

## Adding key in the SSH Agent

We will use ssh-agent to securely save your passphrase. Make sure the ssh-agent is running and add your key (the -K option is to store the passphrase in your key chain, macOS only).

```bash
eval "$(ssh-agent -s)" && ssh-add -K ~/.ssh/<personal_key>
```

## Adding keys in the SSH config

We need to edit our `config` file inside the `~/.ssh` directory, if not created then we need to create one using the `touch ~/.ssh/config` command.

Here, in the `Host`, which is a string, used when the ssh command connects to the remote host. For example, you can use `github.com-work` or `github.com-personal` in the Host.

```plaintext
Host <adding-host-name-1>
HostName github.com
User git
IdentityFile ~/.ssh/<ssh-key-for-account-1>

Host <adding-host-name-2>
HostName github.com
User git
IdentityFile ~/.ssh/<ssh-key-for-account-2>
```

## Configuring public key in the VCS

For this setup, just copy your public key, which will be in this format of `<created_key>.pub`. You can do this using the following commands.

macOS

```shell
tr -d '\n' < ~/.ssh/<created_key>.pub | pbcopy
```

Linux

```shell
xclip -sel clip < ~/.ssh/<created_key>.pub
```

Windows

```shell
cat ~/.ssh/<created_key>.pub | clip
```

And simply just go to the `SSH and GPG keys` in the settings in GitHub.

Click on `New SSH key` and simply paste the public key under the `Key` and give it a meaningful title. Also, for the `Key type` leave it as `Authentication Key` only.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1737887168896/e1cafae3-8708-4346-9be6-e78947112041.png align="center")

## Structuring the workspace

Now comes the interesting parts of setting up the workspace, so we’ll be dividing our main folders under two sub directories `personal` and `work` in the `/home/<user>` directory.

In the `/home/<User>` directory will be our `.gitconfig` file, so the whole structure would look like in the format given below:

```shell
/home/<User>/
    |__.gitconfig
    |__work/
    |__personal/
```

Then, on top of this we’ll be override the `gitconfig` in the sub directories.

```shell
/home/<User>/
|__.gitconfig
|__work/
     |_.gitconfig.work
|__personal/
    |_.gitconfig.pers
```

## Populating git configs

Here, an example of the `gitconfig` in the sub directories. First, we’ll have the `.gitconfig.pers` example:

```plaintext
# ~/personal/.gitconfig.pers
 
[user]
email = <personal-email>
name = <commit-author>
 
[github]
user = "<Personal-GitHub-username>"
 
[core]
sshCommand = "ssh -i ~/.ssh/<private-key-for-personal-account>"
```

Second, for the `.gitconfig.work` example:

```plaintext
# ~/work/.gitconfig.work
 
[user]
email = <work-email>
name = <commit-author>
 
[github]
user = "<Work-GitHub-username>"
 
[core]
sshCommand = "ssh -i ~/.ssh/<private-key-for-work-account>"
```

We can specify which SSH key to use by explicitly defining the SSH command with the `-i` option, which allows us to select an identity file or key. This approach provides greater control, especially in scenarios where you have multiple accounts on the same host (e.g., GitHub). Using an SSH config file to assign different keys would require creating custom prefixes (like [`first.github.com`](http://first.github.com)) and remembering to use them whenever cloning a repository for that account. This method can feel less flexible and somewhat inconvenient.

Now, comes the global `.gitconfig` example:

```plaintext
# ~/.gitconfig
 
[includeIf "gitdir:~/personal/"] # include for all .git projects under personnal/ 
    path = ~/personal/.gitconfig-pers
 
[includeIf "gitdir:~/work/"]
    path = ~/work/.gitconfig-work
 
[core]
excludesfile = ~/.gitignore      # valid everywhere
```

Since many settings, such as your name or editor preferences, are likely to be shared between configurations, you can define them here.

Keep in mind that Git has three levels of configuration that override each other: system, global, and local (project-specific). You can find more details about configuration options [here](https://git-scm.com/book/en/v2/Customizing-Git-Git-Configuration).

To see a list of **all config options**

```shell
man git-config
```

To check your **activated config options**

```shell
git config --list
```

## Repeat steps for other accounts

You can repeat the all these steps, for other git accounts, and configure those account safely and securely.

## Testing our configs

Now, we can easily clone, push, add origin of multiple GitHub accounts.

But, first to test the `ssh` connection from Account-A, we can run the following command:

```bash
ssh -T git@github.com-personal
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1737889891896/7f9cce5d-8b50-4462-b879-0b6ca9e1033f.png align="center")

As you can see, in the above image response from `github`.

### Cloning repository

In order to clone a repository, we have to modify the clone command a little bit for connecting to our configured Host.

```bash
git clone git@github.com-<personal or work>:deepanshu-rawat6/deepanshu-rawat6.git
```

We need to add `-<personal or work>` depending on our configured Host.

## Add origin

```bash
git remote add origin git@<repository.domain.com>:<username>/<repo_name>.git
```

# Final Words

So this was it for this blog everyone, I hope you’re able to configure multiple Git and GitHub accounts in your local machine.

If you liked this blog, a like would be appreciated, and do let me know areas of improvement. Also, if you want me to blog about some other services of AWS or DevOps, do let me know in the comments or you can reach me on Twitter.

See you on the next blog✨

%[https://media.giphy.com/media/v1.Y2lkPTc5MGI3NjExZXU0NnMxb292bXE0d3k0czl3OGpybDd6MTVlN3ZudGg3aDMzNXQ1ciZlcD12MV9naWZzX3NlYXJjaCZjdD1n/ZBVhKIDgts1eHYdT7u/giphy.gif]