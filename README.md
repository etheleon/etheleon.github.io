
<!-- vim-markdown-toc GFM -->

* [Introduction](#introduction)
  * [Getting Started](#getting-started)
  * [Misc](#misc)

<!-- vim-markdown-toc -->

# Introduction

To check if the post works run the following

```
export JEKYLL_VERSION=3.8
docker run --rm \
  --volume=$PWD:/srv/jekyll \
  -p 35729:35729 -p 4000:4000 \
  -it jekyll/builder:$JEKYLL_VERSION \
  jekyll build
```

## Getting Started

So Simple takes advantage of Sass and data files to make customizing easier and requires Jekyll 3.x.

## Misc

To learn how to install and use this theme check out the [Setup Guide](http://mmistakes.github.io/so-simple-theme/theme-setup/) for more information.

