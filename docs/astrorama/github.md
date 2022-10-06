---
layout: page
title: GitHub repositories
menubar: docs_menu
---

All of Astrorama's projects are stored under [github.com/astrorama](https://github.com/astrorama/).

For some of them, as Alexandria, Elements and SourceXtractor++,
there are private mirrors under [Euclid's GitLab](https://gitlab.euclid-sgs.uk/).

However, both Phosphoros and PhosphorosCore GitHub repositories are read-only
mirrors of those in GitLab.

## Special repositories

### [astrorama.github.io](https://github.com/astrorama/astrorama.github.io)

Contains this documentation. See [About this page](/docs/astrorama/astrorama.github.io)
for more information.

### *-feedstock

`*-feedstock` repositories contain the recipe needed to build
and [deploy conda packages](/docs/astrorama/conda/) for the astrorama software and some of
their dependencies.

See [Releases / Conda](/docs/releases/conda/) for a step-by-step guide on releasing
tagged and development version of our software.

* [`levmar-feedstock`](https://github.com/astrorama/levmar-feedstock) [Levenberg-Marquardt nonlinear least squares algorithms in C/C++](http://users.ics.forth.gr/~lourakis/levmar/)
* [`elements-feedstock`](https://github.com/astrorama/elements-feedstock) The base build framework used by the rest of the components.
* [`alexandria-feedstock`](https://github.com/astrorama/alexandria-feedstock) A collection of functions used by SourceXtractor++ and Phosphoros.
* [`phosphoroscore-feedstock`](https://github.com/astrorama/phosphoroscore-feedstock) Core functionality of the template-fitting software Phosphoros
* [`phosphoros-feedstock`](https://github.com/astrorama/phosphoros-feedstock) Graphical User Interface and command line utilities built on top of Phosphoros
* [`sourcextractor-feedstock`](https://github.com/astrorama/sourcextractor-feedstock) Extraction of catalogs from astronomical images

### [.github](https://github.com/astrorama/.github)

It is a special, organization-wide, configuration repository. For instance,
it contains the organization's [README.md](https://github.com/astrorama/.github/blob/master/profile/README.md)
file, which is displayed when viewing the organization's profile:

![README.md content on astrorama's profile](/docs/astrorama/readme.png)

It also contains [workflow-templates](https://github.com/astrorama/.github/tree/master/workflow-templates),
which can be used to setup actions on new repositories: Actions > New workflow > RPM - Repository (By astrorama).
See ["Creating starter workflows for your organization"](https://docs.github.com/en/actions/using-workflows/creating-starter-workflows-for-your-organization)
for more information about workflow templates.

### [actions](https://github.com/astrorama/actions)

Under this repository we store a collection of [GitHub actions](https://docs.github.com/en/actions/learn-github-actions/understanding-github-actions#actions)
shared by astrorama's repositories.
At the time of writing, we have 4 actions. We describe them in the order they
are expected to be used:

1. `elements-project` Extracts project name and version from the main
   `CMakeLists.txt` file and stores them under `project` and `version` outputs
   respectively.
2. `setup-dependencies` Automatically installs the project requirements using
   the dependencies specified on the `CMakeLists.txt` file (under `USE`),
   and the list provided on the file specified by `dependency-list`.
3. `elements-build-rpm` Builds the RPMs.
4. `upload-rpm` Uploads the RPMs to `http://repository.astro.unige.ch/euclid/`,
   under a directory that [depends on the build type](https://github.com/astrorama/actions/blob/v3.1/upload-rpm/upload-rpm.sh#L42)
   (i.e., pull, branch or tags go to `devel/pull/#number`, `devel/branch` or the
   root directory respectively).

Here you can see a typical excerpt for the workflow file:

```yaml
# .github/workflows/main.yml

steps:
  - name: Checkout
    uses: actions/checkout@v3
  - name: Get package name and version
    id: package-version
    uses: astrorama/actions/elements-project@v3.1
  - name: Install dependencies
    uses: astrorama/actions/setup-dependencies@v3.1
    with:
      dependency-list: .github/workflows/dependencies.txt
  - name: Build
    id: build
    uses: astrorama/actions/elements-build-rpm@v3.1
  - name: Upload RPM to repository
    uses: astrorama/actions/upload-rpm@v3.1
    if: ${{ '{{' }} github.repository_owner == 'astrorama' }}
    env:
      REPOSITORY_USER: ${{ '{{' }} secrets.REPOSITORY_USER }}
      REPOSITORY_PASSWORD: ${{ '{{' }} secrets.REPOSITORY_PASSWORD }}
      REPOSITORY_KEY: ${{ '{{' }} secrets.REPOSITORY_KEY }}
    with:
      rpm-dir: ${{ '{{' }} steps.build.outputs.rpm-dir }}
      srpm-dir: ${{ '{{' }} steps.build.outputs.srpm-dir }}
```

## Mirroring of Phosphoros

PhosphorosCore and Phosphoros are read-only on GitHub, except for the
user [sdc-ch](https://github.com/sdc-ch). [Mirroring works with GitLab](https://docs.gitlab.com/ee/user/project/repository/mirror/)
pushing to GitHub automatically on push, using an [SSH key generated by
GitLab](https://docs.gitlab.com/ee/user/project/repository/mirror/#get-your-ssh-public-key).
These public keys are added to sdc-ch.
