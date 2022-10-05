---
layout: page
title: Relasing on Conda
menubar: docs_menu
---

If you check the [Astrorama GitHub Organization](https://github.com/astrorama)
you will see there is a `*-feedstock` repository per component. Each one will
have one master branch for the production release, and additional branches -
as develop - if we want to distribute pre-releases for testing.
Nothing much changes either way, just make sure to use the proper branch
depending on what you want to build.

<article class="message is-warning">
  <div class="message-body" markdown="1">
Normally `master` points to the latest release. Let's assume it to be
Alexandria 2.26.0. If we need to patch, or rebuild, Alexandria 2.25.0,
we can create a new branch called `release/2.25.0`.
  </div>
</article>

Builds are automated, so every time there is a push, [Azure](https://dev.azure.com/astrorama/feedstock-builds/_build)
will pick the changes, build and upload to [Anaconda.org](https://anaconda.org)
the new package (unless you add `[skip ci]` to the commit message!).

<article class="message is-info">
  <div class="message-body" markdown="1">
In the Astrorama organization, you may notice there is a repository called
[`levmar-feedstock`](https://github.com/astrorama/levmar-feedstock).
Levmar is not part of `conda-forge` so we need to build it ourselves.
  </div>
</article>

## Install tooling for `conda`

First, we need to make sure we have all the required utilities
for making a release. We assume `conda` and `git` are at least installed.
Let's activate `conda` if it isn't already:

```bash
# Replace bash with your shell of choice: e.g., shell.zsh
eval "$(${HOME}/miniconda3/bin/conda shell.bash hook)"
```

We consider good practice to have the tooling on its own environment.
Let's call it `build`. We need to install, at the very least,
`conda-smithy`. It will also pull `conda-build`, which is a useful tool:

```bash
conda create -n build conda-smithy
conda activate build
```

<article class="message is-info">
  <div class="message-body" markdown="1">
Remember to enable `conda-forge` in your conda configuration, or pass `-c conda-forge`
when creating the environment!
  </div>
</article>

## `*-feedstock` content

Let's pick Phosphoros as an example. First, make sure you have the `feedstock`
checked out locally

```bash
git clone git@github.com:astrorama/phosphoros-feedstock.git
cd phosphoros-feedstock
```

Let's do an `ls -a1` an examine the content:

<pre><font color="#3465A4"><b>.azure-pipelines</b></font>
<font color="#3465A4"><b>.ci_support</b></font>
<font color="#3465A4"><b>.circleci</b></font>
<font color="#3465A4"><b>.git</b></font>
.gitattributes
<font color="#3465A4"><b>.github</b></font>
.gitignore
<font color="#3465A4"><b>.scripts</b></font>
azure-pipelines.yml
<font color="#4E9A06"><b>build-locally.py</b></font>
<font color="#3465A4"><b>build_artifacts</b></font>
conda-forge.yml
<font color="#C4A000"><u style="text-decoration-style:single"><b>README.md</b></u></font>
<font color="#3465A4"><b>recipe</b></font>
</pre>

Most of these files are handled by `conda-smithy` and we do not need to worry
about them. We care about `conda-forge.yml` and everything inside `recipe`.
For the former, we refer to the [official documentation](https://conda-forge.org/docs/maintainer/conda_forge_yml.html).

For the later, we can also examine its content:

<pre>build.sh
conda_build_config.yaml
cross-linux.cmake
meta.yaml
<font color="#75507B"><b>phosphoros_64.png</b></font>
trigger_compiler_id.patch
</pre>

The main file is [`meta.yaml`](https://docs.conda.io/projects/conda-build/en/latest/resources/define-metadata.html).
It defines the package name, version, build number, origin for the sources,
build and run-time dependencies, test command and general metadata.

`build.sh` contains the build instructions (i.e. invoke `cmake` with the right
flags). `conda_build_config.yaml` lists the channels used to build (`channel_sources`)
and the channel where to upload the result `channel_targets`.
`*.patch` files are applied to the sources before configuration and build.

In the particular case of Phosphoros, we define an "app" that can be run from
the Anaconda launcher. Its icon is `phosphoros_64.png`.

## Releasing

Now everything is setup, we can try to do a release. First, let's make sure
everything is up to date:

```bash
git checkout develop && git pull
git checkout master && git pull
```

Let's suppose we are about to release Phosphoros 1.4.0, and that the building
of the [`develop` branch is successful](https://dev.azure.com/astrorama/feedstock-builds/_build?definitionId=11).
We can modify directly the files under `master`, or we can do first a merge from
`develop` so the dependencies are aligned:

```bash
git merge develop
```

Now, we need to modify `meta.yaml`, and update the head of the file:

```yaml
{% raw %}
{% set name = "Phosphoros" %}
{% set version = "1.4.0" %}       # Must align with a released tag
{% set build_number = 1 %}        # Start at 1 for a new release

package:
    name: {{ name|lower }}
    version: {{ version }}

source:
    git_rev: {{ version }}        # Make sure we are checking out the tag!
    git_url: https://github.com/astrorama/Phosphoros.git
    patches:
      - trigger_compiler_id.patch # Review if the patches are still needed

build:
    number: {{ build_number }}{% endraw %}
```

Once the head is is right, cross-check the dependencies:

```yaml
host:
    - python
    - boost-cpp
    - phosphoroscore ==1.4.0     # Need to depend on a released (and built) version!
    - qt >=5.12,<6
    - cfitsio >=3.470
    - CCfits >=2.5
    - lxml >=4.5
    - pybind11 >=2.6
```

<article class="message is-success">
  <div class="message-body" markdown="1">
*SIDE NOTE:* On non-leaf packages (such as Alexandria and Element) you will see
under `run_exports` under the build section:

```yaml
{% raw %}
build:
    number: {{ build_number }}
    run_exports:
      - {{ pin_subpackage('alexandria', max_pin='x.x.x') }}{% endraw %}
```

This tells conda that anything built against Alexandria X.Y.Z **strictly requires**
Alexandria X.Y.Z when installing. This removes the need of pinning the dependency
on the dependent package.
  </div>
</article>

Then, verify that `conda_build_config.yaml` tells conda to install
the dependencies **from the `main` tag only**. Otherwise, we may build a release
depending on a development built:

```yaml
channel_sources:
  - astrorama,conda-forge,defaults

channel_targets:
  - astrorama main
```

Finally, we need to refresh all the files handled by `conda-smithy`:

```bash
conda smithy rerender
```

Generally this is enough to do a release. Depending on the changes, you
*may* need to modify `build.sh` to pass appropriate flags to `cmake`.
Once all is done, just `git add`, `git commit`, and `git push`. It
is advisable to also create and push a tag (i.e. `1.4.0-1`).

<article class="message is-info">
  <div class="message-body" markdown="1">
You can test the build locally with `conda build`

```bash
conda build -c astrorama -c conda-forge -m .ci_support/linux_64_python3.9.____cpython.yaml  recipe/
```

Note, however, that we need to manually specify the channels, and the CI
file generated by `conda-smithy`.
  </div>
</article>

## Rebuilding

From time to time, Conda Forge may upgrade the compiler version for osx (clang),
for Linux (gcc) or both. Typically, this can break a release build since the
(Element-based) dependencies were build with an older compiler than the
just-released software. When this happens, everything will have to be re-built
in order:

1. elements-feedstock
2. alexandria-feedstock
3. sourcextractor-feedstock / phosphoroscore-feedstock -> phosphoros-feedstock

For rebuilding, we can start directly on `master`, and not bother with
the merge, nor the version bump. We just need to:

1. Bump the build number
2. Run `conda smithy rerender`

We commit and push the changes, create a new tag (i.e. `2.25.0-2`)
and push it too. Azure will pick the changes and rebuild everything. Once
a component is done, we can follow with the dependent packages.

<article class="message is-warning">
  <div class="message-body" markdown="1">
If the build number is not changed. The upload will be skipped if the same
combination of version and build number already exists.
  </div>
</article>
