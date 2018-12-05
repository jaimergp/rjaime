---
title: "Tutorial: Remote Data (Work from home, pt. 1)"
date: 2018-12-03T21:37:10+01:00
draft: true
---

This is the first part of the tutorial series [How I work from home](/posts/work-from-home), in which I explain how to access your data remotely via different ways. In this part, I talk about remote data access via `ssh` and `sshfs`, tunneling included.

<!--more-->

> __Part 1 - Table of Contents__
>
> 1. [Access remote data on demand with sshfs](#access-remote-data-on-demand-with-sshfs)
> 2. [Get access through a gateway](#get-access-through-a-gateway)
> 3. [From Bash aliases to the SSH config file](#from-bash-aliases-to-the-ssh-config-file)
> 4. [SSH keys](#bonus-ssh-keys)

# Access remote data on demand with sshfs

You probably know that you can connect to a remote computer using the `ssh` command. If the computer is exposed outside the local network, then you can simply grab its IP or domain name and do `ssh [-p <port>] <user>@<REMOTE> ` and you are in! Then, if you want to copy some files to your home laptop, you use `scp` like this (from your laptop): `scp [-P <port>] <user>@<REMOTE>:<path/to/your/file> .`

This is normally fine and sometimes inevitable, but you will have to wait until the download finishes to read the contents. With large files and slow connections, this can be slow and inefficient. However, if the files you want to read support streamed access, you won't need to download it fully!

In Linux, you can _mount_ the remote path in a local access point thanks to `FUSE` and `sshfs`. This means that the remote data will be magically available in your computer and will be only downloaded when it is needed, on demand.

First, install `sshfs` on your computer. Depending on your Linux distribution, you will use a different package manager:

```bash
# Ubuntu
$ sudo apt install sshfs

# Arch Linux
$ sudo pacman -S sshfs
```

Then, create a local directory to be used as the mount point. You can use the name you want.

```bash
$ mkdir ~/mnt
```

Finally, use `sshfs` to mount the remote path to the recently created directory:

```bash
$ sshfs <user>@<REMOTE>:<path/to/the/remote/directory> ~/mnt
```

You can now access `~/mnt` and it will display the contents of the remote directory you have chosen, both in the terminal and graphical explorer. Depending on the number of items in the directory and your connection speed, it can take a few seconds to first display some contents. You can open files, edit them and create new ones as if they were local files. That's the magic of FUSE!

For example:

```bash
$ cd ~/mnt
$ ls -alh
$ less README.md
$ echo "Example text" > new_document.txt
```

When you are done, you will be able to _unmount_ the remote path using:

```bash
$ fusermount3 -u ~/mnt
```

# Get access through a gateway

Most of the local computers in research or industry environments are behind a firewall that prevents direct access and you have to jump over a gateway server. This is, first you `ssh` into the gateway, and then you `ssh` into your PC.

In ideal conditions:

```
[HOME] --------------(ssh)---------------> [REMOTE]
```

With a gateway server:

```
                                          |
                                          |
[HOME] ---(ssh)---> [GATEWAY] ---(ssh)---> [REMOTE]
                                          |
                                          | firewall
```

> **Disclaimer**
>
> - This guide provides some hints to workaround limitations in network access provided in your workspace (academic or industrial). Ask your IT team first if they are happy about it!
> - SSH tunneling can get heavy in the gateway. Use it responsibly.


Let's assign hostnames, users and custom ports to each computer for further reference (notice how the username matches the hostname's first letter).

| Computer | Network address     | SSH port | Username | Public access |
|----------|---------------------|----------|----------|---------------|
| HOME     | (not available)     | 22       | henry    | No            |
| GATEWAY  | gateway.example.org | 22522    | gerard   | Yes           |
| REMOTE   | remote.example.org  | 22       | robert   | No            |

In order to reach `REMOTE` from `HOME`, your current approach probably involves two separate commands:

```bash
# In HOME, connect to GATEWAY using custom PORT
$ ssh -p 22522 gerard@gateway.example.org
# Now, from GATEWAY, connect to REMOTE
$ ssh robert@remote.example.org
```

Can we do it in one command? Yes! In several ways, depending on your OpenSSH version (check it with `ssh -V`). I will list them from newer (preferred) to older builds.

__OpenSSH 7.3+__ - With `-J`

```bash
$ ssh -J gerard@gateway.example.org:22522 robert@remote.example.org
```

__OpenSSH 5.4 - 7.3__ - With `ProxyCommand`

```bash
$ ssh -o ProxyCommand="ssh -W %h:%p gerard@gateway.example.org" robert@remote.example.org
```

__OpenSSH before 5.4__ - With `ssh -t ... ssh`

```bash
$ ssh -t -p 22522 gerard@gateway.example.org ssh robert@remote.example.org
```

## SSHFS through a gateway

The cleanest way is to use `ProxyJump` again, thanks to the `-o` argument:

```bash
$ sshfs robert@remote.example.org:<path/in/remote/pc> ~/mnt -o ProxyJump="gerard@gateway.example.org:22522"
#                                                     \____/
#                                                local mount point
```

Alternatively, you can create a `ssh` tunnel (_aka_ the [-NfL combo](https://explainshell.com/explain?cmd=ssh+-NfL)) and then `sshfs` from the forwarded port:

```bash
# First, create the ssh tunnel to local port 2222
$ ssh -NfL 2222:remote.example.org:22 gerard@gateway.example.org -p 22522
#  local port_/ \__remote addr+port_/ \____gateway address_____/    \__gateway port

# Now, REMOTE is accessible through localhost at port 2222
$ sshfs -p 2222 localhost:<path/in/remote/pc> ~/mnt
#       \_localhost:2222_/
#     (redirects to REMOTE!)
```

Unmounting it's the same as without gateways:

```bash
$ fusermount3 -u ~/mnt
```

# From Bash aliases to the SSH config file

If you need to type these commands once in a while, it's fine, but you are probably thinking of creating Bash aliases for each command, right? Well, that's OK, but you might want to consider using [`~/.ssh/config`](https://www.cyberciti.biz/faq/create-ssh-config-file-on-linux-unix/).

This file can list all your common SSH connections together with their command-line options in a tidy way. Also, they will be available to other `ssh`-related commands, like `scp` or `sshfs`, and you will be able to use TAB-complete!

Let's say you had this `alias` in your `~/.bashrc` so you can connect to `gateway.example.org` by simply typing `gateway`:

```bash
alias gateway="ssh -p 22522 gerard@gateway.example.org"
```

The (cleaner) `~/.ssh/config` equivalent would be:

```
Host gateway
    Hostname gateway.example.org
    User gerard
    Port 22522
```

And you connect like this:

```bash
$ ssh gateway
```

It's four keystrokes longer, but now it will also work with `scp`, `sshfs` and so on!

```bash
# Now you can stop worrying about whether it is -p or -P
$ scp some_file gateway:data/
```

Convinced now? Let's edit the file with `mkdir -p ~/.ssh; vi ~/.ssh/config` and add the previous `Host gateway` block:

    Host gateway
        Hostname gateway.example.org
        User gerard
        Port 22522

Of course, jumps and forwardings are also available. The `-J` equivalent in `ssh_config` speech is `ProxyJump`:

```bash
# openssh 7.3+ supports -J
# We need `ProxyJump`, which support more than one jump!
# Use comma-separated values for that
Host remote
    Hostname remote.example.org
    User robert
    ProxyJump gerard@gateway.example.org:22522
```

Older versions without `-J` (under 7.3) need `ProxyCommand`:

```bash
# Older openssh builds do not support -J
# We need `ProxyCommand` directives with delegated ssh -W
Host remote
    Hostname remote.example.org
    User robert
    ProxyCommand ssh -p 22522 gerard@gateway.example.org -W %h:%p

# Before openssh 5.4 use intermediate nc proxy
Host remote
    Hostname remote.example.org
    User robert
    ProxyCommand /usr/bin/nc -X connect -x gateway.example.org:22522 %h %p
```

With these entries defined, we now have an alias `remote` that grants direct access to `remote.example.org`, with gateway jumps and everything! We can now go and simply use it with all ssh-related commands.

```bash
$ ssh remote
# And also other ssh related commands!
$ scp myfile remote:some/directory/
$ sshfs remote:<path/to/directory> ~/mnt
```

For SSH tunneling, he `-NfL` combo has no exact translation. You can only specify the `-L` part with `LocalForward`:

```
Host remote2222
    Hostname remote.example.org
    User robert
    ProxyJump gerard@gateway.example.org:22522
    LocalForward 2222 localhost:22
```

So you will have to manually provide the `-Nf` part at runtime:

```bash
$ ssh -Nf remote2222
```

The `~/.ssh/config` file is incredibly useful! Check the [official docs](https://linux.die.net/man/5/ssh_config) for full details on all the available options (wildcards, compression, X-forwarding, and so on).

I like to have persistent, compressed SSH connections that do not hang over some minutes. Since we can specify wildcard hosts to match all possible connections, we can add these lines at the top of the file:

    Host *
        ServerAliveInterval 15
        ServerAliveCountMax 3
        Compression yes


# Bonus: SSH keys

For a better access experience, I recommend [configuring SSH keys for login](https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys--2) instead of password-based. That way, you won't need to type the passwords, especially on bastioned hosts! You will have to do it for all connections involved; ie, `HOME -> GATEWAY` and `GATEWAY -> REMOTE`.

### Direct access to remote PC

In your home PC:

```bash
$ ssh-keygen -t rsa
# Follow the assistant
$ ssh-copy-id robert@remote.example.org
```
### Access through gateway

In your home PC:

```bash
$ ssh-keygen -t rsa
# Follow the assistant
$ ssh-copy-id gerard@gateway.example.org -p 22522
```

Then, `ssh` into the gateway (you won't need the password now, probably) and repeat for the lab PC:

```bash
$ ssh-keygen -t rsa
# Follow the assistant
$ ssh-copy-id robert@remote.example.org
```

Done! No more passwords!

# Wrapping up

That's all for now. With this first post I hope I have covered how to access your remote data even if it is behind a gateway server. Next part will be devoted to [remote data _analysis_](/tutorials/remote-data-analysis)), so the files stay where they are.

__All parts__

1. [Remote file access](/tutorials/remote-data-access)
2. [Remote data analysis](/tutorials/remote-data-analysis)
3. [Remote graphical interfaces and desktop](/tutorials/remote-graphical-access)