---
title: Trickz
position: 1
badge: By @luastan
description: Some Tips and tricks!
---

<tricks-animated-logo></tricks-animated-logo>



## Adding content


### Live editing

It is usefull to edit the content of tricks with docker as you can see changes in realtime. First pull the image from the repo:

```bash
docker pull "{{ image gcr.io/luastans-trickz/ssr-tricks }}"
```

If you can acces the source code, you can also build the image from source:

```bash
git clone "https://github.com/luastan/tricks.git" tricks
docker build -t "{{ image tricks }}" tricks
```

Then you can mount any folder with markdown on it. The following command uses the `mounting-point` directory as a markdown source and also forwards the port `lport` on your host to the container:


```bash
docker run -v "{{ mounting-point /home/$USER/projects/tricks-content }}:/app/content" -p "{{ lport 8080 }}:8080" --user root --entrypoint /usr/local/bin/yarn -it "{{ image gcr.io/luastans-trickz/ssr-tricks }}" dev
```

This will expose the web app at <code>localhost:<smart-variable variable="lport">8080</smart-variable></code> generated from markdown files located at <code><smart-variable variable="mounting-point">/home/$USER/projects/tricks-content</smart-variable></code>. Keep in mind that if you are using windows, you have to mount a wsl directory to launch the container. Otherwise hot reloading wont work. 

My recomendation is to clone the content to any directory you want in your WSL home and use the [Remote - WSL](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-wsl) extension for Visual Studio Code or directly acces with your `explorer.exe` the `\\wsl$` route where you can find the filesystem for all your distros.


### Usage


#### General usage
Trickz uses Nuxt Content under the hood so you can follow the [nuxt/content documentation](https://content.nuxtjs.org/writing) on how to use every markdown feature the tech has.


#### Code snippets & Smart variables


<pre class="language-md fancy-scrollbar"><code class="language-md">## Code snippet examples

 * Language and filename are optional
 * You can add smart variables to code using curly braces:

```language[filename]
<span class="text-gray-500 italic">curl --header "Authorization: key=<span class="font-semibold">{{ variable-name defaultValue }}</span>" \
    --header Content-Type:"application/json" \
    https://fcm.googleapis.com/fcm/send
```
The <i>smart-variable</i> component can be used anywhere to display smart variable values:

<span class="font-semibold">&lt;smart-variable variable="variable-name"&gt;default value&lt;/smart-variable&gt;</span> 

Put <span class="font-semibold">&lt;smart-variable variable="var-name"&gt;default&lt;/smart-variable&gt;</span> anywhere.
</code></pre>


#### HTML & CSS

You can freely add HTML tags to markdown files, and even use Vue components. 


#### Images
Place the images on the same path your markdown file is using a unique filename. The image will be available at `/filename`. Take the following code as an example:

```markdown
![{{ image-caption Image Caption }}](/{{ image-filename sample.png }})
```


