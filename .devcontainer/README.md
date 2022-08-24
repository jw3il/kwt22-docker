# KWT22 Docker Workshop - Docker with VS Code

This directory contains a sample configuration file for [Docker development with VS Code](https://code.visualstudio.com/docs/remote/containers).

## Prerequisites

You need
* Docker
* VS Code
* The [Remote Development Extension Pack](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.vscode-remote-extensionpack)

## Building the Development Container

Now, you can execute the command `Remote-Containers: Open Folder in Container` in VS Code (e.g. by pressing F1 or Control+Shift+P) and open the root directory of this repository (not `.devcontainer/`).

With the provided `.devcontainer/devcontainer.json` file, VS Code should directly set up the container and your development environment, including the Python extension for editing the code.

## Note on Docker Binds and Linux Permissions

On Linux, files that are created by root in a mounted directory inside the docker container are also owned by root on the host system if you do not run rootless docker.
This is unfortunate, if you later want to edit the files outside of the container.

A simple solution is to run the docker deamon as a non-root user with [rootless docker](https://docs.docker.com/engine/security/rootless/).
Alternatively, one possible workaround is to [create and select a user that shares the same user id (and group id) as the user on your host system](https://code.visualstudio.com/remote/advancedcontainers/add-nonroot-user).

The `Dockerfile` in `.devcontainer/` is similar to the solution of step 4, but we have added commands to create a `flask` user with the standard uid/gid of the first non-root user on a linux system. This user is then selected in `devcontainer.json`.
If you have multiple users, you can call `id $(whoami)` to get your uid and gid and modify the build args in `devcontainer.json` accordingly.

## Running your Project

You can now open a terminal inside your development container with the command `Terminal: Create New Terminal` and then run your application with

```
flask run --host=0.0.0.0 --port=5000
```
