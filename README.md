
<!-- vim-markdown-toc GFM -->

* [Introduction](#introduction)
  * [Getting Started](#getting-started)
  * [Misc](#misc)

<!-- vim-markdown-toc -->

# Introduction

To check if the post works run the following

```
export JEKYLL_VERSION=3.8
export JEKYLL_GITHUB_TOKEN=<token>
docker run --rm \
  --name blog \
  -e JEKYLL_VERSION=$JEKYLL_VERSION \
  -e JEKYLL_GITHUB_TOKEN=$JEKYLL_GITHUB_TOKEN \
  --volume=$PWD:/srv/jekyll \
  -p 35729:35729 -p 4000:4000 -p 9001:9001 \
  -it jekyll/builder:$JEKYLL_VERSION \
  jekyll serve --livereload```

## Getting Started

So Simple takes advantage of Sass and data files to make customizing easier and requires Jekyll 3.x.

## Misc

To learn how to install and use this theme check out the [Setup Guide](http://mmistakes.github.io/so-simple-theme/theme-setup/) for more information.


