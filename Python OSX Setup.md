---
aliases: 
publish: 
---
%%
date:: [[2024-09-14]]
parent:: 
%%
# [[Python OSX Setup]]

I prefer not to use brew to install and manage python as it can change versions when it's a dependancy of of brew installed package. Switching between versions may be required therefore it can break scripts. Also pip the python package manager prevents conflicts when trying to install python packages with pip when using brew for sytsem package management. I recommend:
- [Install Python with Pipx](https://mac.install.guide/python/pipx) for scripts and utilities
- [Install Python with Rye](https://mac.install.guide/python/install-rye) for programming

#### Rye for easy python venv and package management

##### Managing Python Versions
Rye uses toolchains to refer to python versions. Toolchains can be fetched from 2 locations or can be customised.
```
# install a toolchain
rye fetch
# list installed
rye toolchain list
# list downloadable
rye toolchain list --include-downloadable
# pin project to version
rye pin 3
```
##### Managing Python Project Packages

```bash
rye add <package name>
rye sync
```
##### Create Python Projects
```
mkdir myproject
cd myproject
rye pin 3
rye init
rye status
```
#### Notes

Shell initialisation files:

- `~/.Xprofile`: This file is ideal for commands that should be executed only when the terminal window is opened, such as setting the `$PATH` environment variable.
-  `~/.Xshrc`: This file is best for adjusting the appearance and behavior of the shell, such as setting command aliases and adjusting the shell prompt.
#### References
- [Install Python with Rye · Mac Install Guide · 2024](https://mac.install.guide/python/install-rye)