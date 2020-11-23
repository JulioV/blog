---
date: "2020-11-22"
tags: ["web", "hugo"]
title: "Quick Blog Style Update"
---

This is an update to this [post](../personalizing-my-blogs-theme/). 

I set the default font to Wotfard and the default mono font to FiraCode, reduced the font size and increased the line height:

```css
@font-face {
    font-family: 'Wotfard';
    src: url('{{ "css/fonts/wotfard-regular-webfont.woff2" | absURL }}') format('woff2');
    font-weight: regular;
    font-style: normal;
    font-display: swap;
}

@font-face {
    font-family: 'FiraCode';
    src: url('{{ "css/fonts/FiraCode-Regular.woff2" | absURL }}') format('woff2');
    font-weight: regular;
    font-style: normal;
    font-display: swap;
}

html {
    font-size: calc(1em + 0.23vw);
    font-family: 'Wotfard', Helvetica Neue, sans-serif;
    line-height: 2.0;
    color: #111;
    background-color: #fefefe;
}

h1,
h2,
h3,
h4 {
    font-family: Wotfard, serif;
    line-height: 1.125;
    letter-spacing: 2px;
}

code {
    font-family:  FiraCode, Consolas, Monaco, 'Andale Mono', 'Ubuntu Mono', monospace;
    font-size: 0.9em;
    background-color: #eee;
    border-radius: 3px;
    padding: 0 3px;
    color: #004085;
}
```

I added color to links so they are easier to find

```css
main#main  a {
    color: #3f7fa6;
}
```

I removed the bookmark icon from the posts' title (also delete the SVG element from `layouts/_default/single.html`)

```css
/* Pattern lists */
.patterns-list {
    list-style-type: square;
}
```
