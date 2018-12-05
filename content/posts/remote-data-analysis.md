---
title: "Tutorial: Remote Data Access & Analysis (pt. 2)"
date: 2018-12-03T21:37:45+01:00
draft: true
---

Doing computational research often means generating lots of data that needs to be processed and analyzed. In certain areas of molecular modelling, file sizes often go well into the gigabytes or even terabytes. As a result, downloading the full dataset to work on it locally is not only cumbersome, but inefficient. Are there any alternatives? Of course!

<!--more-->

---

In this tutorial series, I will talk about two alternative methods:

1. Working with remote data over SSH(FS) [[part 1](/posts/remote-data-access)]
2. Using Jupyter Notebooks for remote data analysis [part 2, this one]

The former is more general purpose but eventually incurs downloading the data anyway, while the latter does not download any raw data: only the results of the analysis, which is often way less.

> __Table of Contents__
>
> 1. [Install Jupyter Notebook](#01-install-jupyter-notebook)
> 2. [Forward the server connection](#02-forward-the-server-connection)
> 3. [Connect from your browser](#03-connect)

# Remote data analysis with Jupyter Notebook

You can use the methods listed in [pt. 1](/posts/remote-data-access) to access any files you want remotely, but in the end it will incur downloading something. If you simply intend to analyse the data (interactively or not), you might just want to run the analysis remotely as well. You won't have to download anything, and the pipeline will have faster access to the files.

> **Notice**: Take into account that the data processing will run on the remote computer. If this is a shared computer and you run numerically intensive processes, all the other users will notice the performance loss. In that case, consider scripting the interactive analysis in an automated way so you can submit it to a calculation node.

## 01: Install Jupyter Notebook

[Jupyter Notebooks](http://jupyter.org/) are language-agnostic documents composed of _cells_ that allow you to run code interactively while documenting the process. They were created for Python (IPython era) but now admit more languages (Julia, R, Scala...). Behind the scenes, they are JSON files that are rendered by a web application. This means that you access them through the browser, which is perfect for _remote_ access. I mean, that's what browsers were designed for!

Since I mostly use Python for data analysis, I will install Jupyter along a Python environment with `conda`. `conda` is the package manager of the [Anaconda Python distribution for data science](https://www.anaconda.com/), freely available. Anaconda is distributed in two flavours:

- [Anaconda Distribution](https://www.anaconda.com/download/#linux): The full pack with hundreds of packages for data science, Jupyter included.
- [Miniconda](https://conda.io/miniconda.html): A stripped down version with only the basics (Python + conda, mainly), which you can then use to install what you want.

I normally prefer a smaller installation size, so choose Miniconda and then install Jupyter in the REMOTE computer. You don't need it locally (you will just connect to a webserver). Instructions for Miniconda:

```bash
# Connect to remote PC (jump through GATEWAY if necessary)
$ ssh robert@remote.example.org
# Download Miniconda Python 3.7 for Linux 64bit
$ wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
# Install Miniconda
$ bash Miniconda*.sh
# Configure .bashrc
$ echo ". ~/miniconda/etc/default.d/conda.sh" >> ~/.bashrc
$ source ~/.bashrc
# Activate conda base environment and install Jupyter
$ conda activate
$ conda install jupyter notebook ipython
```

Now, you can run Jupyter Notebook in the remote PC. Use the `--no-browser` flag to skip the automatic `$BROWSER` launch.

```bash
# If not already activated
$ conda activate
# Run Jupyter server
$ jupyter notebook --no-browser
```

You will see something like this in the output:

```
[I 16:43:24.055 NotebookApp] Serving notebooks from local directory: /home/user/data
[I 16:43:24.056 NotebookApp] The Jupyter Notebook is running at:
[I 16:43:24.056 NotebookApp] http://localhost:8888/?token=fbccf41667a91f907253cb653654651356623741d947
[I 16:43:24.056 NotebookApp] Use Control-C to stop this server and shut down all kernels (twice to skip confirmation).
[C 16:43:24.056 NotebookApp]

    Copy/paste this URL into your browser when you connect for the first time,
    to login with a token:
        http://localhost:8888/?token=fbccf41667a91f907253cb653654651356623741d947
```

If you `Ctrl+Click` on that URL now, a new tab will open in the browser, but it will fail to connect. We have to fix that by forwarding the connection to our home PC.

You need to keep this process running, so don't close the `ssh` connection! If you really need to do that, consider using `screen` or `tmux` to create a virtual session before launching the Jupyter server. Remember to copy the token code in that case!

## 02: Forward the server connection

At this point, the Jupyter server will be running, but we cannot access it from our home PC. We need to forward the remote IP and port (`8888` by default) to our `localhost` address with SSH tunneling, _aka_ the [-Nfl combo](https://explainshell.com/explain?cmd=ssh+-NfL).

If you have direct access to the remote PC running the Jupyter server, it is easy. We will use the same port at both sides for simplicity:

```bash
$ ssh -NfL 8888:localhost:8888 robert@remote.example.org
```

If you have to jump through a gateway server, you can use `-J` like before and it will behave as expected:

```bash
$ ssh -J gerard@gateway.example.org:22522 -NfL 8888:localhost:8888 robert@remote.example.org
```

Without it, you can do it with two forwardings:

```bash
# First connection let us bypass the gateway
$ ssh -NfL 2222:remote.example.org:22 gerard@gateway.example.org -p 22522
# We can now access REMOTE through localhost at port 2222!
# Bind the remote 8888 to our local 8888.
# Notice how the username matches REMOTE, not HOME
$ ssh -NfL 8888:localhost:8888 -p 2222 robert@localhost
```

## 03: Connect!

If the 2nd step was successful, you can now open a new tab in your browser and enter `http://localhost:8888`. It will ask for a token, which should be listed in the Jupyter server session you created in the 1st step. In fact, you can `Ctrl+Click` in that link to open the webapp directly in your local browser. It will open now!

You can create a new Notebook by using the dropdown menu in the top right corner and begin your data analysis. You will be able to type your code without lag because the commands will only be sent to the remote PC when you execute the cell! Remember that you can also run shell commands by prepending a exclamation mark (`!ls -alh`).
