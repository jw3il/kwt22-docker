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

## Running your Project

You can now open a terminal inside your development container with the command `Terminal: Create New Terminal` and then run your application with

```
flask run --host=0.0.0.0 --port=5000
```
