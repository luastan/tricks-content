---
title: Trickz
position: 1
badge: By @luastan
description: Some Tips and tricks!
---

<tricks-animated-logo-glitch></tricks-animated-logo-glitch>

## Adding content

### Live editing

It is usefull to edit the content of tricks with docker as you can see changes in realtime. First pull the image from the repo:

```bash
docker pull "{{ image gcr.io/luastans-trickz/ssr-tricks }}"
```

If you can acces the source code, you can also build the image from source:

```bash
docker build -t "{{ image tricks }}" "https://github.com/luastan/tricks.git#master"
```


Then you can mount any folder with markdown on it. The following command uses the `mounting-point` directory as a markdown source and also forwards the port `lport` on your host to the container:


```bash
docker run --rm -v "{{ mounting-point /home/$USER/projects/tricks-content }}:/app/content" -p "{{ lport 8080 }}:8080" --user root --entrypoint /usr/local/bin/yarn -it "{{ image gcr.io/luastans-trickz/ssr-tricks }}" dev
```

This will expose the web app at <code>localhost:<smart-variable variable="lport" default-value="8080"></smart-variable></code> generated from markdown files located at <code><smart-variable variable="mounting-point" default-value="/home/$USER/projects/tricks-content"></smart-variable></code>. Keep in mind that if you are using windows, you have to mount a wsl directory to launch the container. Otherwise hot reloading wont work. 

My recomendation is to clone the content to any directory you want in your WSL home and use the [Remote - WSL](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-wsl) extension for Visual Studio Code or directly acces with your `explorer.exe` the `\\wsl$` route where you can find the filesystem for all your distros.

### Usage

Learn more about how to write content for Trickz on the [Adding Content](/tricks/add-content) page.