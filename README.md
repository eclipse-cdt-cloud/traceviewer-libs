# Trace Viewer Libraries (git subtree)

This repository contains the trace viewer libraries and minimal infrastructure to maintain them in source code form:

- `traceviewer-base` and 
- `traceviewer-react-components`

The libraries are meant to provide generally useful building blocks for making a Trace Viewer application based on the Trace Server Protocol (TSP). They are used in at least two components:

- The VSCode Trace Viewer extension ([repo](https://github.com/eclipse-cdt-cloud/vscode-trace-extension))
- The Theia Trace Viewer extension ([repo](https://github.com/eclipse-cdt-cloud/theia-trace-extension))

It's possible to update the libraries using this repo here, but it might be preferable to update the local version, that's a copy of their source code, imported as a git subtree. In other words, this repo here is best used as for collaboration, where to share improvements to the libraries done in a local subtree, or to pull improvements done by someone else. 

## Why split the trace viewer libraries in their own repo?

For historical reasons, the libraries were initially co-located with the Theia Trace Viewer and consumed in source form (built along with it) during development, making co-development easy. They were published to npm as needed, along with the extension. In the VSCode Trace Viewer extension, the libraries were consumed from npm at build time and bundled with the published extension (to the Microsoft Marketplace and Open VSX registry). This made it a lot more challenging to co-develop the VSCode Trace Viewer and the `traceviewer` libraries.

We considered several ways to improve upon this situation, with the following goals:

- Trace Viewer app/extension feature development testing and upstreaming: make it as easy as possible for all consumer of the libraries. Specially when the libraries need to be updated along with the consuming component. 
- Collaboration on the libraries: make it as simple as possible, so that consumers can share improvements and new features they added
- Preserve git authorship contribution history even if the libraries are moved to a new repository

The solution we picked is to use a `git subtree`. This involved splitting-out the trace viewer libraries from their original development location with `git subtree split`, without using the `--squash` option, so history is preserved. This repo here is the result of that operation, plus some minimal added infrastructure.

In a separate step, this repo can be used in `vscode-trace-extension`, to "import" a local version of the libraries, as a git subtree. 

## How it was done

The `traceviewer-*` libraries were split-out from `theia-trace-extension` repo like so:

```bash
cd theia-trace-extension
git checkout master
# update local branch to latest master
git pull origin master
# clean all local files
git clean -ffdx


# split content of the "packages" folder (where the traceviewer
# libraries are). Put the resulting content on a branch called 
# "traceviewer-libs-branch"
git subtree split --prefix=packages --branch traceviewer-libs-branch

```

Note: by itself, the above does not remove the original libraries from the main branch of the `theia-trace-extension` repository. 

## TODO

### ~~Push the `traceviewer-libs-branch` git subtree to its own repository~~

Below is how it was done for validation purposes. This can be repeated with an official repository, under the "eclipse-cdt-cloud" GitHub organization

1) Open a ticket on the Eclipse Foundation Gitlab and ask for an empty repo to be created, named "traceviewer-libs"
2) When done, push the latest version of the subtree branch to it (note: use the correct repo URL - below used a committer's GitHub):

```bash
git remote add traceviewer-libs git@github.com:marcdumais-work/traceviewer-libs.git
git push traceviewer-libs traceviewer-libs-branch:master

```

Depending how the repo it setup, it may be necessary to push to a PR branch and merge to master after a review.

## Use the subtree in another repo

### How to use the `traceviewer-libs` subtree in another repository

This is an example on how to add the subtree to a repo, replacing that repo consuming the
libraries from npm. 

```bash
cd vscode-trace-extension
# when available, use the official "eclipse-cdt-cloud" subtree repo instead
git remote add traceviewer-libs git@github.com:marcdumais-work/traceviewer-libs.git
git subtree add --prefix=traceviewer-libs traceviewer-libs master --squash

```

In the root package.json, add the libraries in the "workspaces" array:
"workspaces": [
    [...]
    "local-libs/traceviewer-libs/*"
    [...]
}

More changes/tweaks might be necessary.


### Push local changes made to the subtree towards the subtree repo

```bash
git subtree push -p <subtree folder> <subtree remote repo> <remote review branch>

```

### Pulling latest changes from the subtree repo into the local subtree

```bash
# make sure your local master is up-to-date:
git checkout master && git pull origin master
git branch update-subtree && git checkout update-subtree
git subtree pull --prefix=<local subtree folder> <subtree remote repo> master --squash

e.g.: 
git subtree pull --prefix=local-libs/traceviewer-react traceviewer-react master --squash

# push update branch and create a PR from it, have it reviewed and merged ASAP:
git push origin update-subtree

```

