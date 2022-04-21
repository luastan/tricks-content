---
title: Adding content
description: How to add content to the tricks repo
badge: Trickz
position: 1
---

The following is a brief guide on how to write documentation for Trickz. There's no crazy setup or knowledge needed as the same markdown you would use for Github/Gitlab or other viewers will work out of the box. 

## Markdown

As stated before plain vanilla markdown works just as expected.

### Page properties

Trickz will parse markdown properties defined at the beggining of the file. The syntax would be like a `yaml` file between two lines containing just `---`. Just like in the Gitlab wiki and most Gitbooks or git-based headless CMS, **the only required property is `title`** which defines the title for the page:

```markdown
---
title: {{ title Title for your page }}
---

The content goes here
```

The following is a list with a description of all available properties:

- `title` _(required)_: The title from the page. Will be displayed at the top of the page as well as on the sidebar.
- `description`: A brief description of the page that will appear under the page title.
- `badge`: A higlighted badge/keyword/tag to display on top of your page with the title.
- `position`: An integer that will be used to order the pages on the sidebar.

The following is a code snippet for the current page using every available property

```markdown
---
title: {{ title Adding content }}
description: {{ description How to add content to the tricks repo }}
badge: {{ badge Trickz }}
position: {{ position 1 }}
---

The content goes here
```

> Markdown editors will: ingnore this section (VS Code), use the title property and hide the rest (Gitlab's wiki) or display a table with each property and value (Github's markdown preview)

### Images

Just place it where you feel to. You can reference it relatively from where the page is or speficy the full route to it. If your markdown editor or Github/Gitlab shows the image on the render preview, Trickz will do too.

As an example lets say you are editing `/sample/page1.md` and want to reference `/sample/image1.jpg`. Since both files are on the same directory you have the next 2 options:

```markdown
![Image Caption](/sample/image1.jpg)
![Image Caption](image1.jpg)
```

If your page has a lot of images you might want to create a directory to have a cleaner repo. Following the previous example one could create the `/sample/page1-images` directory and move the images there. Intuitivelly the images could be referenced as follows:

```markdown
![Image Caption](/sample/page1-images/image1.jpg)
![Image Caption](page1-images/image1.jpg)
```

### Table of contents

It gets automatically generated from your titles. The only thing to take in mind is that it will start from the `##` headers, as the `#` is considered to be the page title you defined with the `title` property.
It also shows an _"Edit on github"_ link that references the Github editor on the corresponding markdown file for the current page.

### Links to titles

Headers are automatically assigned an id consisting of the title in lowercase removing special characters an changing spaces with slashes. A link to the current section would be:

```markdown
[Link text](/tricks/add-content#links-to-titles)
```

Any editor with markdown completion will suggest and autocomplete the link for you.

## Vue components

Things start getting interesting. Tricks can use predefined Vue components, which adds huge versatility in the long-run as more components are added.

### Smart Tabs

Very simple component

```markdown
<smart-tabs variable="{{ variable-name tool-docker-vs-cli }}" :tabs="{'{{ tab-a-id docker }}': '{{ tab-a-title Docker }}', '{{ tab-b-id cli }}': '{{ tab-b-title Command line }}'}">
<template v-slot:{{ tab-a-id docker }}>

- Markdown content for tab A

</template>
<template v-slot:{{ tab-b-id cli }}>

- Markdown content for tab B

</template>
</smart-tabs>
```

Which would look somewhat like this:

<smart-tabs variable="tool-docker-vs-cli" :tabs="{'docker': 'Docker', 'cli': 'Command line'}">
<template v-slot:docker>

- Markdown content for tab A

</template>
<template v-slot:cli>

- Markdown content for tab B

</template>
</smart-tabs>

Remember to leave a blank line between HTML code and markdown code when writing between HTML tags or the parser will identify everything as HTML and skip your markdown.

### Smart variables

This component displays the value for a _variable_ that is shared with every page and component and saved on your browser local storage. The code for displaying <smart-variable variable="var" default-value="this"></smart-variable> would be:

```markdown
<smart-variable variable="{{ variable-name var }}" default-value="{{ variable-value this }}"></smart-variable>
```

As a trick you can allow click to copy wrapping the component like <code><smart-variable variable="var" default-value="this"></smart-variable></code> with the `<code></code>` tag as follows:

```markdown
<code><smart-variable variable="{{ variable-name var }}" default-value="{{ variable-value this }}"></smart-variable></code>
```

The main purpose for this component is to be used on code snippets, where a much more simplified template syntax can also be used in flavour of the verbose tag form of the component. More on that [below](/tricks/add-content#smart-variables-1)  

## Code snippets

Code snippets are automatically highlighted with [Prism.js](https://prismjs.com/), and have a bunch of enhancements. The following would be a fully fledged code block (play with it: hover it, use the copy button, change the values, delete the values, type really long values...):

```http[sqli_example.req]
GET /search?q={{ search no sqli here } url-all } HTTP/1.1
Host: {{ target vulnerable.net }}
Cookie: userinfo={{ cookie admin=0 } b64, url }; session=n20zBuYgGlRee
User-Agent: Mozilla/5.0 (X11; Linux x86_64)
```

### Smart variables

Smart variables can be used on code snippets. When doing so, an input will be added bellow to edit the content of each unique variable used.

It is also possible to use a simplified syntax where you could display smart variables with just `{{ variable-name default variable value }}`. **It just requires spaces separating the variable name and the variable value**. With that you would replace the, previously showed, entire component tag with just <code>{{ <smart-variable variable="variable-name" default-value="var"></smart-variable> <smart-variable variable="variable-value" default-value="this"></smart-variable> }}</code>.

Take the following curl snippet as an example:

<pre class="language-md fancy-scrollbar"><code class="language-md">
```bash[api_key.sh]
<span class="text-gray-500 dark:text-neutral-500 italic">curl --header "Authorization: key=<strong>{{ key some-key }}</strong>" \
    --header Content-Type:"application/json" \
    https://fcm.googleapis.com/fcm/send
```</code></pre>

On Trickz, it would render as follows:

```bash[api_key.sh]
curl --header "Authorization: key={{ key some-key }}" \
    --header Content-Type:"application/json" \
    https://fcm.googleapis.com/fcm/send
```

#### Filters

Filters can also be used to perform transformations for the saved values. The syntax consists of just adding them between the last curly braces `{{ var value } filter1, filter2 }`, and are applied in the order they are read from left to right. Take the following XSS payload as an example:

<pre class="language-md fancy-scrollbar"><code class="language-md">
```html
<span class="text-gray-500 dark:text-neutral-500 italic">&quot;&gt;&lt;input onfocus&equals;eval&lpar;atob&lpar;this&period;id&rpar;&rpar; id&equals;<strong>&lcub;&lcub; xss-payload alert&lpar;1&rpar; &rcub; b64 &rcub;</strong> autofocus&gt;</span>
```</code></pre>

On Trickz, it would render as follows:

```html
"><input onfocus=eval(atob(this.id)) id={{ xss-payload alert(1) } b64 } autofocus>
```

For the moment, the following filters are available:

- `b64`: Encodes the value in base 64.
- `url`: URL encodes the value.
- `url-all`: URL encodes every special character from the value.
- `escape-4-json`: Escapes double quotes `"` as `\"`.
- `lower`: Transforms into lowercase.
- `upper`: Transforms into uppercase.
- `html`: Encodes value into HTML entities.
