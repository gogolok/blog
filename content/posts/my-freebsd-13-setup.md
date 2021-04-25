---
title: "My Freebsd 13 Setup"
date: 2021-04-25
tags:
- FreeBSD
---

From time to time I'm doing some stuff on FreeBSD.
I still like it for its simplicity (I'm old and Linux is progressing way to
fast ;-).

I'm keeping some notes how to set up FreeBSD manually in this blog post.
For newer versions of FreeBSD I'll create a new post.

**Disadvantages**:
- manual setup

## Update to Latest Stable Version

After a fresh install, we first apply latest updates to the current FreeBSD
version.

``` shell
# freebsd-update fetch install
# shutdown -r now
```

```shell
# pkg
The package management tool is not yet installed on your system.
Do you want to fetch and install it now? [y/N]:
```

## Sudo

```shell
# pkg install sudo
```

```shell
# vi /usr/local/etc/sudoers
%wheel ALL=(ALL) ALL
```

## Shell

```shell
# pkg install bash zsh
```

## New User And Default Shell

```shell
# adduser
Username: gogo
Full name:
Uid (Leave empty for default):
Login group [gogo]: wheel
Login group is wheel. Invite gogo into other groups? []: wheel
Login class [default]:
Shell (sh csh tcsh bash rbash zsh rzsh nologin) [sh]: zsh
Home directory [/home/gogo]:
Home directory permissions (Leave empty for default):
Use password-based authentication? [yes]
Use an empty password? (yes/no) [no]:
Use a random password? (yes/no) [no]:
Enter password:
Enter password again:
Lock out the account after creation? [no]:
Username   : gogo
Password   : *****
Full name  :
uid        : 1001
Class      :
Groups     : wheel
Home       : /home/gogo
Home Mode  :
Shell      : /usr/local/bin/zsh
Locked     : no
Ok? (yes/no): yes
adduser: INFO: Successfully added (gogo) to the user database.
Add another user? (yes/no): no
Goodbye!
```

**Please log in to the new user from now on.**

## General Packages

More comfort:

```shell
$ sudo pkg install vim-console tmux
```

For development in general:

```shell
$ sudo pkg install git
```

## Ruby

### rbenv

Compilation dependencies:

```shell
pkg install gmake
```

```shell
$ git clone https://github.com/rbenv/rbenv.git ~/.rbenv
$ git clone https://github.com/sstephenson/ruby-build.git ~/.rbenv/plugins/ruby-build
```

```shell
$ echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.zshenv
$ echo 'eval "$(rbenv init -)"' >> ~/.zshenv
```

**Afterwards you need to relogin to your user account.**

Now we can install a rather new Ruby version:

```shell
$ rbenv install --list

$ rbenv install 2.7.3
$ rbenv global 2.7.3
```
