# Setting up SSH keys

_Note: this guide is written for macOS, but we do intend to support other platforms!_

SSH is one of the most important network utilities you can learn. It is like the copper pipe of network plumbing. The `ssh` terminal command lets you easily and securely login to remote servers, but SSH is also a transport used by `git` and `rsync`.

This document is meant to help new users generate SSH keys, or to configure an existing key pair to work with Smol Data servers, and to commit to our GitHub repos. We will be using the command line terminal, so open up Terminal.app (located in Applications/Utilities).

## The .ssh folder

SSH keys and their configuration files live in a hidden folder in your home directory, in a folder called `.ssh`. Let's begin by making that folder if it doesn't exist already.

```
mkdir -p ~/.ssh
```

Next we will go into the `.ssh` folder where we can continue the setup process.

```
cd ~/.ssh
```

In macOS, any folder that starts with a `.` is hidden from the Finder. If you're curious to look inside, you can use the `open` command to look at it with the Finder.

```
open ~/.ssh
```

## Look inside the .ssh folder

Let's look around in the folder to see what's there already. Get a directory listing of the `.ssh` folder.

```
cd ~/.ssh
ls -l
```

If you've never set up SSH keys before, your directory will probably be empty. If you find files that look like `id_rsa` and `id_rsa.pub` there, you can skip the next step generating keys since you already have them.

`id_rsa` and `id_rsa.pub` are the default names, but you may decide you want to use another name like `smol` and `smol.pub`. It is a good security habit to keep credentials for different services compartmentalized, but going with the default is fine to start out with. You can always generate new keys later.

## Generate a key pair

Let's create a new pair of SSH keys.

```
ssh-keygen
```

Accept the default name `id_rsa`. You don't _have_ to set a passphrase on the key, but if you do it will mean anyone who gains access to the files won't be able to use your key pair to login as you. ("Passphrase" means the same thing as "password," in case you aren't accustomed to seeing it written it that way.)

When you finish, you should see two new files in the `.ssh` folder: `id_rsa` and `id_rsa.pub`.

```
ls -l
```

## What files are in there?

Here are some of the files you may find in the `.ssh` folder:

* `[key]` and `[key].pub` key pairs (default `[key]` name is `id_rsa`, but it can vary)
* `config` holds settings for different servers
* `known_hosts` keeps a list of SSH fingerprints of servers you've visited
* `authorized_keys` contains a list of public keys authorized to login on this account

## Private key vs. public key

Each SSH key pair has a _private_ and _public_ file (the public one has a `.pub` suffix). The **private key** is the one you need to keep secret, it's basically like a passphrase stored in a file. You never want to share this file with anyone, or upload it to the Internet. The **public key** is _not_ secret, you will need to share it with GitHub and store it on each server you want to login to.

## File permissions

It's important that file permissions are set up properly, or your keys won't work. The `.ssh` folder itself should have `700` permissions (you have full access, nobody else gets any access) and the private key should have `600` permissions (you have read/write access, nobody else gets any access).

```
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_rsa
```

## Example server config

The `~/.ssh/config` file is a very handy way to make SSH easier to work with. You don't have one by default, so you'll need to create a new `config` file yourself.

Inside the file, each server gets a set of configurations that are applied when you invoke the `ssh` command. Each setting under `Host [name]` gets applied when you use the command `ssh [name]`.

Here is an example of a server entry for `pinto` (a shortcut for `pinto.smoldata.org`):

```
Host pinto
	Hostname pinto.smoldata.org
	User dphiffer
	IdentityFile ~/.ssh/id_rsa
	ForwardAgent yes
	IdentitiesOnly yes
```

Now I can easily login to `pinto.smoldata.org` by typing the command `ssh pinto`. Here is what each line does:

* `Host pinto` establishes a new set of rules under the shortcut name `pinto`.
* `Hostname pinto.smoldata.org` is the full hostname used to login.
* `User dphiffer` sets which username to use to login to the server (you will want to change this to your own username).
* `IdentityFile ~/.ssh/id_rsa` assigns your private key as the login credential instead of requiring a passphrase.
* `ForwardAgent yes` sets up SSH key forwarding, which makes your private key usable from an SSH session on the server.
* `IdentitiesOnly yes` sets SSH to not send other keys, which could result in `Too many authentication failures` errors.

## Set up SSH key forwarding

That last configuration is important for being able to commit your work to GitHub from a server SSH session. There is one additional command you need to run for that to work properly.

```
ssh-add -K ~/.ssh/id_rsa
```

For some versions of macOS this command needs to be run every time you reboot your computer. If you find that you mysteriously can't commit code to GitHub from the server, you may just need to re-run the `ssh-add` from the macOS terminal (*not* from the server's SSH session).

## Login to the server

Before we continue, copy the contents of your public key to the clipboard. We're going to use [unix pipes](https://en.wikipedia.org/wiki/Pipeline_%28Unix%29) to send the file contents, printed with the `cat` utility, into the macOS `pbcopy` clipboard tool.

```
cat ~/.ssh/id_rsa.pub | pbcopy
```

The first time you login to a new server you will use your passphrase, and then once you're logged in you'll want to set up your public key to use for subsequent logins. Public key logins make it a lot easier to use SSH, since you don't have to type passphrases in all the time.

Let's login to the server called `pinto` using the settings we used in the `.ssh/config` file.

```
ssh pinto
```

The first time you login you will get prompted for your passphrase. This is the _server passphrase_ and is separate from the one used for macOS or to unlock your private key.

You should see a welcome message like this:

```
       _       _
 _ __ (_)_ __ | |_ ___
| '_ \| | '_ \| __/ _ \
| |_) | | | | | || (_) |
| .__/|_|_| |_|\__\___/
|_|

hi, I am pinto the web server.
Last login: Tue Jan  2 15:11:00 2018 from 8.28.55.30
dphiffer@pinto:~$
```

Your username will be different, but the `dphiffer@pinto:~$` prompt indicates that the commands you type here are getting run on the remote server "pinto" and not on your own computer.

## Set up authorized_keys

Let's make sure there's a `.ssh` folder on the server.

```
mkdir -p .ssh
```

Notice that I omitted the `~/` home path. We are using a _relative path_ to the `.ssh` folder since we're already in the home directory by default.

Next, create a new file called `authorized_keys`.

```
nano authorized_keys
```

Next, paste in your public key from your clipboard. This file can have more than one public key, so long as each one starts on a new line.

```
ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQ...
```

Save (`ctrl-O`) then exit the nano editor (`ctrl-C`). Note: you want to hold down the actual `ctrl` key, not the more common `cmd` key.

## Test out SSH keys

Let's logout of the server and then try logging back in with SSH keys.

```
exit
```

Now you should see your usual macOS prompt. Login to the server again with SSH.

```
ssh pinto
```

You should login to the server without needing to type your passphrase. Or, if you set a passphrase when you generated the private key, type that in now. (It's fine to let the macOS keychain remember your private key login.)

If it _doesn't work_, you may want to try signing in with the "verbose" flag set, which will give you some additional debug info.

```
ssh -v pinto
```
