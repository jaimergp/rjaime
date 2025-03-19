---
title: "Use conda without messing with your shell profiles"
date: 2025-03-19T22:35:40+01:00
---

`conda` famously modifies shell profiles like `.bashrc`, or even the Windows registry, to register a series of shell functions that make it, allegedly, more accessible and user-friendly. Allegedly. After years of using it and contributing to the project, I realized I don't like that design at all and, more importantly: there are alternatives. In this blog post I introduce a prototype subcommand called `conda spawn` and explain why I'm excited about it.

<!--more-->

## When you run `conda`, you are running a shell command

`conda` installers recommend setting up a shell command (which, depending on your platform, will take the form of a shell function or executable script) that can sit between your terminal session and the Python logic powering `conda`.
The shell command is conveniently named `conda`, so most users won't even realize they are using it.

This shell command has several responsibilities:

- It makes `conda` available everywhere, even if the Python entry point cannot be found in PATH.
- It provides the `conda activate` and `deactivate` subcommands, which can modify your shell session without restarting the process.
- Similarly, it can enable auto-activation of a given environment on shell startup.
- In some shells, rehashes the PATH cache (e.g. via `hash -r`) so newly installed executables can be found without restarting the session.

However, all these nice and convenient features come at a complexity and maintenance cost:

- The shell command needs to be initialized in every shell session. This is done via modifications to your shell profile, Windows registry and other mechanisms, depending on your platform and shell. This can incur in performance overheads at shell startup. It's also a potential leftover after incomplete uninstalls.
- The initialization only happens automatically for login shells like `bash -l`. If you run a script and try to call `conda`, suddenly it does not work. This is a source of confusion among many beginners.
- The shell command needs to hardcode the absolute path of the `conda` Python entry point to execute it robustly; it can't rely on PATH entries.
- These system modifications aren't controlled by `conda` packages, so they are tricky to keep updated with newer `conda` releases. Users would need to run `conda init` every now and then.
- The overall shell logic makes `conda` diverge from what otherwise would just be "one more appliction written in Python" and hinders the development experience.

After seeing how `pixi` and `poetry` get away with similar features without modifying anything at the shell level, I wondered: can I make `conda` work without any shell layers? I'm glad to report that the answer is **YES**.

## Introducing the `conda-spawn` plugin

`conda-spawn` is available as an incubated project at `conda-incubator/conda-spawn`. You can install it in your `base` environment with:

```
conda install -n base conda-forge::conda-spawn
```

Once installed, you need to perform some setup steps. At least for now; this might be automated in the future. The steps:

1. Annotate the root of your conda installation with `conda info --root`
2. Run `conda init --reverse` to uninstall the shell modifications and stop using the shell command.
3. Add the directory `<result of step 1>/condabin` to your PATH variable.

From now on, this is your conda workflow:

- No environment is activated by default. To work on an environment, run `conda spawn env-name`. For example, `conda spawn base`.
- When you activate a new environment, a new shell subprocess is launched. To deactivate the environment, you do _not_ run `conda deactivate`. You just exist the subprocess with <kbd>Ctrl</kbd>+<kbd>D</kbd>, `exit`, or your shell equivalent.

There are some advanced options you can check with `conda spawn --help`.

## How does it work?

I was surprised to see it was not that complicated. Most of this is inspired if not stolen from [`poetry-plugin-shell`](https://github.com/python-poetry/poetry-plugin-shell), [`rattler_shell`](https://github.com/conda/rattler/tree/main/crates/rattler_shell) and [`rattler_pty`](https://github.com/conda/rattler/tree/main/crates/rattler_pty).

On Unix shells, it uses pseudo-teletypes (`pty`) to launch new interactive shells and send the initialization commands directly to the input. This is important for shell variables like `PS1`, which control the shell prompt and can't be set from the parent process. On Windows, it simply uses a subprocess because there's no complication around shell prompts and these variables can be set from the parent process just fine. [`shellingham`](https://github.com/sarugaku/shellingham) is used to auto detect which shell needs to be launched.

There are some extra tricks to make sure that the root `conda` remains visible in the newly spawned shell and is not shadowed by a potential `conda` binary in the new environment. A shell function is injected (but only on that process! no shell configuration files touched, promise!) to provide the a forwarder and PATH rehasher.

The rest of the activation logic uses the same code as `conda activate` and `deactivate`.

## Why is this not on conda?

This prototype plugin proves that no shell integration is necessary to use `conda` on a daily basis. I've been doing so since the first release of `conda-spawn` and haven't looked back. I can look at my shell profile and see no remnants of any conda installation, beyond adding a new location to my PATH, which is pretty standard. My shell starts quicker too!

I'd like to integrate this natively in conda so it's part of the default user experience. For now, it's available as a plugin so we can gather feedback from the community. If you have any comments, please open an issue in [`conda-incubator/conda-spawn`](https://github.com/conda-incubator/conda-spawn).