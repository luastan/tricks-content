---
title: Trickz
position: 1
badge: By @luastan
description: Some Tips and tricks!
---

<tricks-animated-logo></tricks-animated-logo>


## Global variables
One of the main features I wanted to build is the possibility of defining global variables. ¿What are those? Go on some of the pages that have content on them and try changing the `collaborator` and `target-domain` variables:

 - [OAuth 2.0](/web/advanced/oauth)
 - [XXE injection](/web/server-side/xxe#exploitation)
 - [Google API keys](/cloud/google/api-keys)

The variables are updated across the whole documentation and saved on your browser's localStorage. 

This results very useful when trying a subset of payloads or snippets against a target. 


## Adding content


### Live editing

It is usefull to edit the content of tricks with docker as you can see changes in realtime. First pull the image from the repo:

```bash
docker pull "{{ image gcr.io/luastans-trickz/ssr-tricks }}"
```

Then you can mount any folder with markdown on it. The following command uses the `mounting-point` directory as a markdown source and also forwards the port `lport` on your host to the container:


```bash
docker run -v "{{ mounting-point /home/luastan/projects/tricks-content }}:/app/content" -p "{{ lport 8080 }}:8080" --user root --entrypoint /usr/local/bin/yarn -it "{{ image gcr.io/luastans-trickz/ssr-tricks }}" dev
```

Keep in mind that if you are using windows, you have to mount a wsl directory to launch the container. Otherwise hot reloading wont work. 

My recomendation is to clone the content to any directory you want in your WSL home and use the [Remote - WSL](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-wsl) extension for Visual Studio Code or directly acces with your `explorer.exe` the `\\wsl$` route where you can find the filesystem for all your distros.
