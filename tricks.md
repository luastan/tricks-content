---
title: Trickz
position: 0
badge: By @luastan
description: Some Tips and tricks!
---

This is just a documentation test, using Nuxt Content. It is generated from vanilla markdown and has support for vue components.



<svg class="w-full" viewBox="0 0 1024 1024" fill="none" xmlns="http://www.w3.org/2000/svg">
<rect width="1024" height="1024" class="fill-gray-900"/>
<path d="M684.803 514V148.016H742.53V514H684.803Z" fill="#FAFAFA"/>
<path d="M388.259 512.91C417.889 512.91 440.973 518.211 457.511 528.814C473.704 539.416 485.073 553.701 491.619 571.666C497.821 589.632 500.922 609.659 500.922 631.748V758.097C500.922 782.248 497.821 803.306 491.619 821.272C485.073 839.237 473.704 853.08 457.511 862.799C440.973 872.813 417.889 877.819 388.259 877.819C362.075 877.819 341.403 873.696 326.244 865.45C310.74 857.203 299.715 845.422 293.168 830.107C286.278 814.792 282.832 796.385 282.832 774.885V746.611H338.13V771.792C338.13 785.046 338.991 796.679 340.714 806.693C342.092 817.001 346.226 824.953 353.117 830.549C360.008 836.145 371.55 838.943 387.743 838.943C404.28 838.943 416.339 835.85 423.919 829.665C431.498 823.775 436.494 815.234 438.906 804.042C440.973 793.145 442.007 780.333 442.007 765.607V623.796C442.007 605.831 440.284 591.546 436.839 580.944C433.393 570.635 427.708 563.273 419.784 558.855C411.86 554.437 401.179 552.228 387.743 552.228C371.894 552.228 360.525 555.173 353.634 561.064C346.743 567.249 342.437 575.642 340.714 586.245C338.991 596.848 338.13 609.218 338.13 623.354V649.085H282.832L282.832 623.354C282.832 601.56 285.933 582.269 292.135 565.481C298.336 548.988 309.017 536.029 324.176 526.605C339.336 517.475 360.697 512.91 388.259 512.91Z" fill="#FAFAFA"/>
<path d="M443.137 512.91L444.359 185.841H439.5V148H525.669C553.011 148 575.627 151.304 593.517 157.911C611.408 164.218 624.572 174.58 633.011 188.996C641.787 203.112 646.176 221.733 646.176 244.859C646.176 258.975 644.488 271.889 641.112 283.602C637.737 295.015 632.336 304.776 624.91 312.885C617.484 320.694 607.695 326.4 595.543 330.004L653.275 512.91V528.5H598.581V512.91L544.91 341.717H500.353L501.919 512.91H443.137ZM500.353 305.227H522.125C538.327 305.227 551.492 303.424 561.619 299.82C571.745 296.216 579.171 290.059 583.897 281.35C588.623 272.64 590.986 260.476 590.986 244.859C590.986 223.535 586.598 208.218 577.821 198.907C569.045 189.296 551.661 184.491 525.669 184.491H500.353V305.227Z" fill="#FAFAFA"/>
<path d="M390.945 877.819L446 849L443.137 512.91H501.919L501.919 685.003L598.674 512.91H653.294L570.064 673.74L666.819 877.819H609.598L528.969 706.627L501.919 748.975L454 854L390.945 877.819Z" fill="#FAFAFA"/>
<path d="M283.248 630.779V185.841H212V148H439.5H447.5V185.841H444.359H342.015V580.725L283.248 630.779Z" fill="#FAFAFA"/>
<path d="M623.819 877.819V841.33L747.593 549.415H631.62V512.926H810V535L679.465 841.33H810V877.819H623.819Z" fill="#FAFAFA"/>
<path d="M443 165H501V703H443V165Z" fill="#FAFAFA"/>
</svg>


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
docker pull "{{ image gcr.io/luastans-tricks/ssr-tricks }}"
```

Then you can mount any folder with markdown on it. The following command uses the `mounting-point` directory as a markdown source and also forwards the port `lport` on your host to the container:


```bash
docker run -v "{{ mounting-point /home/luastan/projects/tricks-content }}:/app/content" -p "{{ lport 8080 }}:8080" --user root --entrypoint /usr/local/bin/yarn -it  "{{ image gcr.io/luastans-tricks/ssr-tricks }}" dev
```

Keep in mind that if you are using windows, you have to mount a wsl directory to launch the container. Otherwise hot reloading wont work. 

My recomendation is to clone the content to any directory you want in your WSL home and use the [Remote - WSL](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-wsl) extension for Visual Studio Code or directly acces with your `explorer.exe` the `\\wsl$` route where you can find the filesystem for all your distros.
