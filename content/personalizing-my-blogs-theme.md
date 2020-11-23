---
date: "2020-04-08"
tags: ["web", "hugo"]
title: "Polishing my blog's appearance and performance"
---

UPDATE. I made more small changes as detailed [here](../quick-blog-style-update/)

When I was setting up this blog with Hugo and Netlify, I found five themes that I liked for their simplicity and aesthetics. I chose [Cupper](https://cupper-hugo-theme.netlify.com/) because it focuses on content, is accessible, posts are grouped by tags and not categories, it supports multiple shortcodes (like notes, warnings, code, among others), it includes minimal javascript, and it provides a dark theme.

- https://cupper-hugo-theme.netlify.com/ (the one I went for)
- https://themes.gohugo.io//theme/harbor/
- https://themes.gohugo.io/theme/kiss/
- https://themes.gohugo.io/theme/hugo-ink/
- https://blog.bespinian.io/

As good as the theme is, I tweaked some things, and this post is a compilation of them for my future reference and for other people that might find them useful. A neat tip is that you don't need to modify the original theme to make changes, instead you can add it as a submodule in the `themes` folder, create the files with the modifications you need in the folders at the top level of your blog and [Hugo will use them first](https://gohugo.io/templates/lookup-order/).

The TL;DR list of changes is below, but you can keep reading for more details:
- Support for more highlighted languages in code snippets by Prism JS
- CSS changes for text readability
- Make the blog's post list its homepage
- Support for static comments using Staticman
- Support for web analytics using GoatCounter
- RSS feed with full posts instead of a short description
- Image, CSS, and JavaScript optimization in Netlify

First, I updated the code highlighting languages supported by Prism JS, the highlighting library used by Cupper. This is the [URL](https://prismjs.com/download.html#themes=prism&languages=markup+css+clike+javascript+bash+git+java+json+latex+markdown+python+r+rest+sql+toml+yaml&plugins=toolbar+copy-to-clipboard) of my configuration that includes Markup, CSS, JS, Bash, Git, Java, JSON, Latex, Markdown, Python, R, SQL, YAML, and TOML. Don't forget you need to update Prism's JS and CSS.

I swapped the original logo for a text title, and made some small CSS adjustments to make the text more readable: decreased the contrast of the dark theme by 15%, changed the general line height, letter spacing of titles, and top margin of paragraphs. 

```css
/* The dark theme settings are a separate style tag in the header*/
.intro-and-nav, .main-and-footer { filter: invert(85%) }
* { background-color: inherit }
img:not([src*=".svg"]), .colors, iframe, .demo-container { filter: invert(85%) }

/* This is the site's main CSS */
html {
    font-size: calc(1em + 0.33vw);
    font-family: Arial, Helvetica Neue, sans-serif;
    line-height: 1.5;
    line-height: 1.8;
    color: #111;
    background-color: #fefefe;
}

h1,
h2,
h3,
h4 {
    font-family: Miriam Libre, serif;
    line-height: 1.125;
    letter-spacing: 2px;
}

p + p {
    margin-top: 1rem;
}

.logo {
    border: 0;
    font-size: 1.5rem;
}
```

I substituted the original homepage for the blog's post list as I want them to be the focus of visitors, and made the about section an external link to my personal website. For the first change, I had to move the theme's post template from `post/single.html` to `_default/single.html`, so all .md files in the `content` folder are rendered with it, and moved `_default/list.html` to `layouts/index.html`, so the original list of posts is now at the homepage. For the second change, I added a conditional to the navigation links rendering to make it open in a new tab:

```liquid
<a href="{{ .URL }}" {{ if $active }}aria-current="page"{{ end }} {{ if eq .Name "About" }}target="_blank"{{ end }}>
```

I added support for Staticman comments instead of Disqus to have a git-backed, lightweight, ethical comment provider. You can read more about the whole process [here](https://jvblog.net/configuring-staticman-hugo/)

I added support for [GoatCounter](https://www.goatcounter.com/) for web analytics instead of Google Analytics for [these reasons](https://plausible.io/blog/remove-google-analytics); [this post](https://mentalpivot.com/ethical-web-analytics-alternatives-google/) by David Papandrew made the search for an alternative easier. GoatCounter gives me the level of detail just right to know what are the most visited posts, referrals, and visitors' platforms, it is GDPR friendly as it does not rely on cookies, just around 1.5Kb, open-source, and free for under 100k pageviews a month. You can support them in [GitHub Sponsors](https://github.com/sponsors/arp242) and [Patreon](https://www.patreon.com/arp242). Installing GC was really easy, all I had to do was to create an account there and add the following script:

```js
<script data-goatcounter="https://MY_SITE.goatcounter.com/count" async src="//gc.zgo.at/count.js"></script>
```

I updated the RSS template to include full posts instead of just descriptions to make them work better with readers like Feedly, which I use a lot. I added [this file](https://github.com/JulioV/blog/commit/81a11583027f5d4b83a00ea2808ab88e87adb731) to `layouts/rss.xml`.

Finally, I activated the asset optimization in Netlify to compress images and minify and bundle CSS and JS files using my `netlify.toml` file. For the latter to work, all their links need to be relative, so I modified the following lines in my templates:

```liquid
<link rel="stylesheet" href="{{ "css/prism.css" | relURL }}" media="none" onload="this.media='all';">
<link rel="stylesheet" type="text/css" href="{{ $styles.RelPermalink }}">

<script src="{{ "js/prism.js" | relURL }}"></script>
<script src="{{ "js/dom-scripts.js" | relURL }}"></script>
```

```toml
// Append this to your netlify.toml
[build.processing]
 skip_processing = false
[build.processing.css]
 bundle = true
 minify = true
[build.processing.js]
 bundle = true
 minify = true
[build.processing.images]
 compress = true
```

All these changes gave the blog a [PageSpeed](https://gtmetrix.com/reports/jvblog.net/XsWRPzvq) score of 100%, Ylow Score of 94% with a Fully Loaded Time of 1.7s and a Total Page Size of 109Kb.