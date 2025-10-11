---
title: 自动补全
weight: 10
description: 为你的 shell 启用 Docker 命令与标志的自动补全
keywords: cli, shell, fish, bash, zsh, completion, options
aliases:
  - /config/completion/
---

可以使用 `docker completion` 命令为 Docker CLI 生成 shell 自动补全脚本。生成后，当你在终端中输入并按下 `<Tab>` 时，可对命令、标志以及 Docker 对象（如容器名、卷名）进行补全。

当前支持为以下 shell 生成补全脚本：

- [Bash](#bash)
- [Zsh](#zsh)
- [fish](#fish)

## Bash

在 Bash 中启用 Docker CLI 补全，需先安装 `bash-completion` 软件包，它提供了用于补全的一系列 Bash 函数。

```bash
# Install using APT:
sudo apt install bash-completion

# Install using Homebrew (Bash version 4 or later):
brew install bash-completion@2
# Homebrew install for older versions of Bash:
brew install bash-completion

# With pacman:
sudo pacman -S bash-completion
```

After installing `bash-completion`, source the script in your shell
configuration file (in this example, `.bashrc`):

```bash
# On Linux:
cat <<EOT >> ~/.bashrc
if [ -f /etc/bash_completion ]; then
    . /etc/bash_completion
fi
EOT

# On macOS / with Homebrew:
cat <<EOT >> ~/.bash_profile
[[ -r "$(brew --prefix)/etc/profile.d/bash_completion.sh" ]] && . "$(brew --prefix)/etc/profile.d/bash_completion.sh"
EOT
```

然后重新加载 shell 配置：

```console
$ source ~/.bashrc
```

现在可以用 `docker completion` 生成 Bash 补全脚本：

```console
$ mkdir -p ~/.local/share/bash-completion/completions
$ docker completion bash > ~/.local/share/bash-completion/completions/docker
```

## Zsh

Zsh 的[补全系统](http://zsh.sourceforge.net/Doc/Release/Completion-System.html)会在补全脚本位于 `FPATH` 路径下时自动启用。

如果你使用 Oh My Zsh，可将脚本放在 `~/.oh-my-zsh/completions` 目录中，无需修改 `~/.zshrc`：

```console
$ mkdir -p ~/.oh-my-zsh/completions
$ docker completion zsh > ~/.oh-my-zsh/completions/_docker
```

若不使用 Oh My Zsh，可将脚本保存到任意目录，并在 `.zshrc` 中将其加入 `FPATH`：

```console
$ mkdir -p ~/.docker/completions
$ docker completion zsh > ~/.docker/completions/_docker
```

```console
$ cat <<"EOT" >> ~/.zshrc
FPATH="$HOME/.docker/completions:$FPATH"
autoload -Uz compinit
compinit
EOT
```

## Fish

fish shell 原生支持[补全系统](https://fishshell.com/docs/current/#tab-completion)。要为 Docker 启用补全，将脚本复制或链接到 fish 的 `completions/` 目录：

```console
$ mkdir -p ~/.config/fish/completions
$ docker completion fish > ~/.config/fish/completions/docker.fish
```
