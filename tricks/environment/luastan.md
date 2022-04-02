---
title: Luastan's Environment
description: This is the environment I use for pentesting/development. It's a basic Windows set-up with WSL 2 and docker. I also configure some certs and stuff for burp and other proxies.
---

## Windows Set-Up

### PowerToys

[PowerToys](https://docs.microsoft.com/en-us/windows/powertoys/) is an amazing set of utilities that greatly boost productivity on windows. Grab the installer from their [GitHub releases](https://aka.ms/installpowertoys).

### Windows Terminal

You can install it from the [Microsoft Store](https://www.microsoft.com/store/productId/9N0DX20HK701). I use it for accessing my WSL distros, Powershell and SSH connections.

### VS Code

Markdown edition:

- docsmsft.docs-markdown
- yzhang.markdown-all-in-one
- DavidAnson.vscode-markdownlint

## WSL

As I said, I use WSL 2 with Arch Linux (Simple distro with a huge talented community and access to a lot of packages easily). I use WSL 2 for Docker support. So, lets start by installing WSL with powershell in an admin prompt:

```powershell
wsl --install
wsl --set-default-version 2
```

This command will enable the required optional components, download the latest Linux kernel, set WSL 2 as your default, and install a Linux distribution for you (Ubuntu by default, see below to change this)[^1].

[^1]: Check the official documentation at: [docs.microsoft.com/en-us/windows/wsl/install](https://docs.microsoft.com/en-us/windows/wsl/install)

I like to keep the ubuntu WSL installation because you never know when you might find an stego challenge that the tool that can resolve it can only be located at the ubuntu repos (Happens more often than I would like).

### Using a diferent directory for WSL

When using WSL for pentesting, your anti-malware solution (Windows Defender and so...) might identify your WSL installations as a threat, as basic web shells and reverse shell payloads are usually treated as malicious. For this reason I like to have a folder for such activities that is marked as an exception for the antivirus. This also allows me to use a different drive and have more control over the files on my system, giving me a better understanding of how I use the space on my drives.

Let's say your WSL pentesting distros are located at <code><smart-variable variable="wsl-location">D:\pentesting\wsl</smart-variable></code>. Here is how to move a distro you already have installed:

Create the directories if you haven't already:

```powershell
New-Item -Path "{{ wsl-location D:\pentesting\wsl }}\{{ distro-name ubuntu }}" -ItemType Directory
```

Now, with a Powershell **admin** prompt, start by finding out the name of the distro you want to move and check if it is running:

```powershell
wsl -l -v --all
```

In case the distro is running you need to stop it:

```powershell
wsl -t "{{ distro-name ubuntu }}"
```

After that you can export the distro to a tar file:

```powershell
wsl --export "{{ distro-name ubuntu }}" "{{ wsl-location D:\pentesting\wsl }}\{{ distro-name ubuntu }}-ex.tar"
```

Now unregister the previous WSL installation, as we will register it back again on the new location:

```powershell
wsl --unregister "{{ distro-name ubuntu }}"
```

Finally import the distro back:

```powershell
wsl --import "{{ distro-name ubuntu }}" "{{ wsl-location D:\pentesting\wsl }}\{{ distro-name ubuntu }}" "{{ wsl-location D:\pentesting\wsl }}\{{ distro-name ubuntu }}-ex.tar"
```

If weverything went right, the distro will show up on the list:

```powershell
wsl -l -v --all
```

Set it as your default WSL distro if you want:

```powershell
wsl -s "{{ distro-name ubuntu }}"
```

Remember that after exporting the distro the default user will be set as **root**, change it with the following command:

```powershell
ubuntu config --default-user {{ wsl-user luastan }}
```

### Arch linux on WSL

You can find the latest insaller release at [Arch WSL releases](https://github.com/yuk7/ArchWSL/releases/latest). You can download it to the distro folder you created before and exctract it. Inside the unzipped folder you will find an executable file (`Arch.exe`) that will install the distro. The name you use for the exe will be the name for your WSL, you can change it if you want:

```powershell
New-Item -Path "{{ wsl-arch-dir D:\pentesting\wsl\arch }}" -ItemType Directory
(New-Object net.webclient).Downloadfile("{{ arch-wsl-zip https://github.com/yuk7/ArchWSL/releases/download/22.3.18.0/arch.zip }}", "{{ wsl-arch-dir D:\pentesting\wsl\arch }}.zip")
Expand-Archive -Path "{{ wsl-arch-dir D:\pentesting\wsl\arch }}.zip" -DestinationPath "{{ wsl-arch-dir D:\pentesting\wsl\arch }}"
Remove-Item "{{ wsl-arch-dir D:\pentesting\wsl\arch }}.zip"
Rename-Item -Path "{{ wsl-arch-dir D:\pentesting\wsl\arch }}\Arch.exe" -NewName "{{ wsl-arch-dir D:\pentesting\wsl\arch }}\{{ arch-wsl-name Arch }}.exe"
{{ wsl-arch-dir D:\pentesting\wsl\arch }}\{{ arch-wsl-name Arch }}.exe
```

After the installation finishes you can open the distro and proceed with the initial Set-Up:

```powershell
{{ wsl-arch-dir D:\pentesting\wsl\arch }}\{{ arch-wsl-name Arch }}.exe
```

On the bash prompt from wsl, setup a password for root and follow the ArchWiki to find help about [Sudo](https://wiki.archlinux.org/index.php/Sudo#Example_entries) and [User and groups](https://wiki.archlinux.org/index.php/Users_and_groups)

```bash
passwd
echo "%wheel ALL=(ALL) ALL" > /etc/sudoers.d/wheel
useradd -m -G wheel -s /bin/bash {{ wsl-user luastan }}
passwd {{ wsl-user luastan }}
exit
```

After that you can set the WSL default user for the one you just created:

```powershell
{{ wsl-arch-dir D:\pentesting\wsl\arch }}\{{ arch-wsl-name Arch }}.exe config --default-user {{ wsl-user luastan }}
```

If the default user has not been changed, reboot the computer or restart the LxssManager in powershell as **admin**:

```powershell
net stop lxssmanager && net start lxssmanager
```

As a last step, update the packages with `sudo pacman -Syu`. **If pacman does not work**, you might have to initialize the keyring to install and update packages:

```bash
sudo pacman-key --init
sudo pacman-key --populate
sudo pacman -Syy archlinux-keyring
sudo pacman -Syu
```

Last but not least, you can mark the Arch distro as my default WSL distro.

```powershell
wsl -s {{ arch-wsl-name Arch }}
```

The command for launching it from the Windows Terminal would be the following:

```powershell
wsl.exe -d {{ arch-wsl-name Arch }}
```

For the starting directory I use `%USERPROFILE%`, this will make the prompt start at the `/mnt/c/Users/MyUsername`

### Docker

Using Docker in Windows with WSL 2 is really easy to install. Just download [Docker Desktop on for Windows](https://docs.docker.com/desktop/windows/install/) and follow the set-up wizard. You might need to restart your computer in order for it to work.

## Linux setup

### Git + Github / Gitlab SSH

First of all install the git and ssh packages

<smart-tabs variable="distro" :tabs="{'arch': 'Arch linux', 'deb': 'Debian / Ubuntu / Kali'}">
<template v-slot:arch>

```bash
sudo pacman -Syu git openssh
```

</template>
<template v-slot:deb>

```bash
sudo apt update
sudo apt install git openssh-client
```

</template>
</smart-tabs>

#### Git gobal configuration

```bash
git config --global user.name "{{ git-username Cool name }}"
git config --global user.email "{{ git-email your_email@example.com }}"
git config --global init.defaultBranch {{ git-default-branch master }}
```

#### Generating a SSH key

```bash
ssh-keygen -t {{ ssh-keygen-algorithm ed25519 }} -C "{{ git-email your_email@example.com }}"
```

Check / Start a new SSH agent on the background

```bash
eval "$(ssh-agent -s)"
```

```bash
ssh-add ~/.ssh/id_{{ ssh-keygen-algorithm ed25519 }}
```

Adding the key to Github / Gitlab:

```bash
cat ~/.ssh/id_{{ ssh-keygen-algorithm ed25519 }}.pub | clip.exe
```

### Yay

> This is only for Arch Linux environments.

Yay (Yet Another Yogurt) is my current favourite AUR helper and the one I use. AUR is a really nice sort-of community-mantained package repository 

```bash
sudo pacman -Syu --needed git base-devel
git clone https://aur.archlinux.org/yay.git /tmp/yay
cd /tmp/yay
makepkg -si
```

### The prompt: ZSH, Oh My Zsh and more

ZSH is a great shell with fantastic plugins and themes. The first step is to install ZSH

<smart-tabs variable="distro" :tabs="{'arch': 'Arch linux', 'deb': 'Debian / Ubuntu / Kali'}">
<template v-slot:arch>

```bash
sudo pacman -Syu zsh
```

</template>
<template v-slot:deb>

```bash
sudo apt update
sudo apt install zsh
```

</template>
</smart-tabs>

You can follow the [oh-my-zsh installation guide](https://ohmyz.sh/#install) which consists basically in running the following command:

<smart-tabs variable="curl-vs-wget" :tabs="{'curl': 'Via curl', 'wget': 'Via wget'}">
<template v-slot:curl>

```bash
sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

</template>
<template v-slot:wget>

```bash
sh -c "$(wget https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh -O -)"
```

</template>
</smart-tabs>

#### Plugins

The plugins are defined at your `~/.zshrc` file. I usually have the following enabled:

```bash
plugins=(
    git
    docker
    docker-compose
    adb
    archlinux
    colored-man-pages
    golang
    compleat
    zsh-autosuggestions
)
```

#### Autocompletion

First of all I usually install [zsh-autosuggestions](https://github.com/zsh-users/zsh-autosuggestions). Just clone the plugin:

```bash
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
```

And add it to your `~/.zshrc`:

```bash
plugins=( 
    # other plugins...
    zsh-autosuggestions
)
```

On top of that I also use the [compleat](https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins/compleat) plugin:

```bash
plugins=( 
    # other plugins...
    compleat
)
```

### Golang

Go recently became my _go-to_ language for developing tools and even scripting. I usually install it right away. You can find the package in most of the distros' repos:

<smart-tabs variable="distro" :tabs="{'arch': 'Arch linux', 'deb': 'Debian / Ubuntu / Kali'}">
<template v-slot:arch>

```bash
sudo pacman -Syu go
```

</template>
<template v-slot:deb>

```bash
sudo apt update
sudo apt install golang
```

</template>
</smart-tabs>

After that, to use the tols/commands/programs you should add the following to your `$PATH`:

```bash
$(go env GOPATH)/bin
```

### Cloud CLIs

I use cloud stuff a lot, specially Google cloud services. I usually have the cloud CLIs installed.

#### Google Cloud CLI

On Arch Linux you can install the tools from [aur/google-cloud-sdk](https://aur.archlinux.org/packages/google-cloud-sdk). For other distros you should follow Google's documentation [^2]. The following are some examples for the distros I use the most:

[^2]: [Installing the gcloud CLI](https://cloud.google.com/sdk/docs/install#deb)

<smart-tabs variable="distro" :tabs="{'arch': 'Arch linux', 'deb': 'Debian / Ubuntu / Kali'}">
<template v-slot:arch>

```bash
yay google-cloud-sdk
```

</template>
<template v-slot:deb>

```bash
sudo apt-get install apt-transport-https ca-certificates gnupg
echo "deb [signed-by=/usr/share/keyrings/cloud.google.gpg] https://packages.cloud.google.com/apt cloud-sdk main" | sudo tee -a /etc/apt/sources.list.d/google-cloud-sdk.list
curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key --keyring /usr/share/keyrings/cloud.google.gpg add -
sudo apt-get update && sudo apt-get install google-cloud-cli
```

</template>
</smart-tabs>

After that you can initialize google cloud with the following command:

```bash
gcloud init --console-only
```

##### Configure access to container registries with Docker

```bash
gcloud auth configure-docker
```
