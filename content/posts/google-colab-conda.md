---
title: "Using Google Colab with Conda"
date: 2019-04-24T14:47:28+02:00
---

[Google Colab(oratory)](https://colab.research.google.com/) is an invaluable resource for data science. You get GPU/TPU computing power while from a Jupyter Notebook frontend running on Google's servers... for free! Surprisingly though, `conda` is not preinstalled in the default configuration. Learn how to fix it!

<!--more-->

---
__TLDR__: Just run these two cells at the beginning of your Colab notebook:

```bash
!wget https://repo.anaconda.com/archive/Anaconda3-5.2.0-Linux-x86_64.sh && bash Anaconda3-5.2.0-Linux-x86_64.sh -bfp /usr/local
```
```python
import sys
sys.path.append('/usr/local/lib/python3.6/site-packages')
```

---
Python [has become](https://insights.stackoverflow.com/trends?tags=java%2Cc%2Cc%2B%2B%2Cpython%2Cc%23%2Cvb.net%2Cjavascript%2Cassembly%2Cphp%2Cperl%2Cruby%2Cvb%2Cswift%2Cr%2Cobjective-c&utm_source=so-owned&utm_medium=blog&utm_campaign=gen-blog&utm_content=blog-link&utm_term=incredible-growth-python) the most popular language in StackOverflow and one could argue that its success is recently due to data science in general and machine learning in particular. Most of the data scientists rely on the [Anaconda distribution](https://www.anaconda.com/) or, at least, its [package manager](https://conda.io/) to install the libraries they need: `conda`.

While most popular projects offer `*.deb` packages and `pip` wheels (both methods officially supported by Google Colab), some are only distributed through `conda` (for example, [OpenMM](http://openmm.org)). However, `conda` is not preinstalled in the Colab environments! The good news is that you can install it manually for each notebook.

## Install Miniconda

Google Colab uses Python 3.6, so we need an Anaconda distribution compiled for that version. Recent builds use Python 3.7, so you have to use Anaconda `v5.2` or Miniconda `v4.5.4`. Choose one below.

### A - Using the full Anaconda distribution

The full Anaconda bundle contains [a huge selection of data science packages](https://docs.anaconda.com/anaconda/packages/py3.6_linux-64/) (650, to be exact) ready to run.

```bash
!wget https://repo.anaconda.com/archive/Anaconda3-5.2.0-Linux-x86_64.sh && bash Anaconda3-5.2.0-Linux-x86_64.sh -bfp /usr/local
```

### B - Using the Miniconda distribution

Miniconda only contains the basics: Python itself, `conda`, `pip` and some libraries. It's up to you to install whatever you need afterwards.

```bash
!wget https://repo.continuum.io/miniconda/Miniconda3-4.5.4-Linux-x86_64.sh && bash Miniconda3-4.5.4-Linux-x86_64.sh -bfp /usr/local
```

## Test the installation

Now, you should be able to run some shell commands to check everything is correct:

```bash
!conda info --all
!conda list
```

## Patch `sys.path` and import your packages

To be able to import the Anaconda packages, you have patch `sys.path` so Python can find the modules:

```python
import sys
sys.path.append('/usr/local/lib/python3.6/site-packages')
```

Now you can just use Python's `import` like usual.


## Install more packages

Of course, you can install more packages if needed. Just remember to use `-y` to avoid interactive prompts and `-q` to remove excessive output. Also, if the package needs `cuda`, make sure it is compiled for `v10`. For example, to install `openmmtools` (which relies on `openmm` and several conda-only packages).

```bash
!conda install -y -q -c conda-forge -c omnia/label/cuda100 -c omnia openmmtools
```

> Solving the conda environment in terms of dependencies can take a while sometimes. In some tests, that last command took 15-20 minutes, but in other cases it finished in 2 minutes. `¯\_(ツ)_/¯`

Check GPU support with `openmm` tests:

```python
import simtk.testInstallation
simtk.testInstallation.main()
```

Expected output is 4 platforms:

```
There are 4 Platforms available:

1 Reference - Successfully computed forces
2 CPU - Successfully computed forces
3 CUDA - Successfully computed forces
4 OpenCL - Successfully computed forces

Median difference in forces between platforms:

Reference vs. CPU: 6.3031e-06
Reference vs. CUDA: 6.73543e-06
CPU vs. CUDA: 7.81258e-07
Reference vs. OpenCL: 6.75426e-06
CPU vs. OpenCL: 8.15821e-07
CUDA vs. OpenCL: 2.17776e-07
```

... And that's it. Maybe in the future Google Colab will bundle `conda` as well and this won't be needed. But as of Apr 2019, this is the way to go!

__Bonus__: [Ready-to-run Miniconda-enabled Notebook](https://gist.github.com/jaimergp/45015e75b4ae5f79a03d24e53b74ac1a).