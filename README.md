<div align="center">
    <h1 align="center">
        <img height="100" alt="codespeck" src="./assets/logo.png" />
        <br/>
        Codespeck
    </h1>
    <p align="center">
        <a href="https://github.com/matteosecli/codespeck/issues"><img alt="GitHub issues" src="https://img.shields.io/github/issues-raw/matteosecli/codespeck?style=flat-square"></a>
        <a href="https://github.com/matteosecli/codespeck/pulls"><img alt="GitHub pull requests" src="https://img.shields.io/github/issues-pr-raw/matteosecli/codespeck?style=flat-square"></a>
        <a href="http://creativecommons.org/licenses/by-sa/4.0/"><img src="https://img.shields.io/badge/License-CC--BY--SA%204.0-lightgrey.svg?color=%234AA4C6&style=flat-square" alt="License"></a>
    </p>
</div>

Serve VS Code (or VSCodium) in a web browser with a single command.

## Why

Serving VS Code with the CLI command `code serve-web` has several limitations, including:
- The server is downloaded only upon first connection.
- Pre-installing extensions is not supported.
- Several options, such as `--disable-workspace-trust` or `--default-folder`, are not available.

Codespeck addresses these limitations by:
- Downloading the web server at setup time, ensuring it's ready when needed.
- Pre-installing extensions and settings via the `devcontainer.json` specification, just like Codespaces.
- Patching the web server to open `README.md` if available, just like Codespaces.
- Forwarding all extra options to `code-server` itself, without the CLI layer.

## Usage

Codespeck is primarily meant to be used in a containerized environment. If your repo has a `.devcontainer/devcontainer.json` with an `image` property, you just run in a container with that image:
```bash
./codespeck --devcontainer .
```
Please note the following:
- You should take care of the proper port forwarding.
- Codespeck will not build any container out of your `devcontainer.json`, let alone support any "feature" property. Building and running the container is your business, and Codespeck will just limit itself to installing extensions and settings, and starting the web server itself.

For additional options, please refer to the `--help` output of the script.

## Demo

Note: you need to have `docker` and `jq` installed on your machine to run the demo.

```bash
git clone https://github.com/microsoft/vscode-remote-try-python.git && cd vscode-remote-try-python
cImage=$(sed 's#//.*##' ./.devcontainer/devcontainer.json | jq -r '.image')
fwPort=$(sed 's#//.*##' ./.devcontainer/devcontainer.json | jq -r '.portsAttributes | keys[0]')
postCmd=$(sed 's#//.*##' ./.devcontainer/devcontainer.json | jq -r '.postCreateCommand')
docker run -u vscode --rm -it -p ${fwPort}:${fwPort} -p 8080:8080 -v "${PWD}:/workspace" -w /workspace \
${cImage} /bin/bash -c "eval '${postCmd}' && curl -fsSl https://raw.githubusercontent.com/matteosecli/codespeck/refs/heads/main/codespeck | bash -s -- --devcontainer . --port 8080 --host 0.0.0.0 --default-folder /workspace --accept-server-license-terms --without-connection-token --disable-workspace-trust"
```