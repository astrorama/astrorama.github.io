---
layout: page
title: Conda channels
menubar: docs_menu
---

# Distributing with Conda

## Introduction

From their documentation, [Conda](https://conda.io/en/latest/) is

> [...] an open source package management system and environment management
  system that runs on Windows, macOS and Linux. Conda quickly installs,
  runs and updates packages and their dependencies.

It's usage is quite similar to `virtualenv` + `pip`. It allows to install a
set of packages in isolated environments, so even packages with conflicting
dependencies can be installed on the same system. Also importantly, it can
be installed, or environments created, by the user without requiring any
special privileges (no root!).

Conda can be installed either via

* [Anaconda](https://www.anaconda.com/distribution/), a complete distribution
  of scientific software, including Python, several scientific packages, and a
  graphic interface (Anaconda Navigator) on MacOSX.
* [Miniconda](https://docs.conda.io/en/latest/miniconda.html), a minimal
  distribution with just Python, the package manager, and their requirements.

<article class="message is-warning">
  <div class="message-body" markdown="1">
**NOTE**: Do not confuse this Anaconda with [Fedora's Anaconda](https://fedoraproject.org/wiki/Anaconda),
which is the one that shows up if you do `dnf search`.
  </div>
</article>

For the following instructions, it does not really matter which one is installed
The instructions apply equally, as both ship with `conda`.

<article class="message is-info">
  <div class="message-body" markdown="1">
[`mamba`](https://mamba.readthedocs.io/en/latest/) is a drop-in re-implementation
of `conda` in C++. It is considerably faster than `conda` when resolving
conflicts.
  </div>
</article>

## Channels

Third parties can build and distribute their own software via "channels".
These channels can be anywhere - be it a local file-system or a web-server -,
but [Anaconda.org](https://anaconda.org) conveniently provides an integrated
platform to manage them. [Astrorama](https://anaconda.org/astrorama) has its own
channel, and our software is there.

Channels need to be activated before any software from them is installed.

Packages can be "labeled", and those with the `main` label are part of the
main channel (say, `astrorama`). Any additional label needs to be explicitly
enabled as an extra channel. As a convention, we use the `develop` label for
packages that have not been released.

Our software has dependencies on packages shipped by [Conda Forge](https://conda-forge.org/)
so that channel *must* also be enabled.

In summary, to install software from Astrorama:

```bash
conda config --add channels conda-forge
conda config --add channels astrorama
```

**Only if** you need software from the development branch:

```bash
conda config --add channels astrorama/label/develop
```

<article class="message is-info">
  <div class="message-body" markdown="1">
It is also possible to specify the channel at installation time

```bash
conda install --channel "astrorama" --channel "conda-forge" package
```
  </div>
</article>

## Installation

Once the channels have been configured, you may install any of our packages.
They can be installed on the default environment - called `base` - but we
strongly recommend to use a separate environment:

* It avoids conflicting or messing up dependencies
* It allows for easier uninstalling
* It allows to install multiple versions at the same time without conflicts,
  and switching easily between them

The first point is *very* important. The software may straight fail to
run otherwise.

You can either create the environment first, activate second, and install third;
but it is simpler, and less error-prone, to do the creation and installation at
once:

```bash
# -n defines the name of the environment!
conda create -n phz phosphoros
# Or
conda create -n phz -c astrorama -c astrorama/label/develop phosphoros
# Or, if you want a specific version
conda create -n phz -c astrorama -c astrorama/label/develop phosphoros==0.11
```

After this is done, you only need to activate the environment to use Phosphoros:

```bash
conda activate phz
Phosphoros GUI
```

To remove the environment:

```bash
# If the environment that you want to remove is active
conda deactivate
conda remove -n phz --all
```

## Distributing

See [Releases / Conda](/docs/releases/conda) for information on how to build and
distribute releases.
