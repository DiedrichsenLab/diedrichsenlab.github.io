---
layout: page
title: Contributing to the WIKI
---

The webpage is build with Jekyll and Fluid.

A good guide to Jeklyll is here:

And an introduction to fluid syntax here:



## Install Jekyll

General instruction:
https://jekyllrb.com/docs/installation/macos/

On my mac
ruby-install ruby 3.1.3

Failed, so I installed ruby with brew

brew install ruby

And added

to .zshrc

export PATH="/usr/local/opt/ruby/bin:$PATH"
source /usr/local/opt/chruby/share/chruby/chruby.sh
source /usr/local/opt/chruby/share/chruby/auto.sh

bundle install

## Local build

```
bundle exec jekyll serve
```

## Deploy

```
bundle exec jekyll build
```
