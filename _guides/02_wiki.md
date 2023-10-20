---
layout: page
title: Contributing to the WIKI
---

The webpage is build with Jekyll (using liquid syntax).
The content is written in markdown - using some liquid syntax.
To contribute and edit pages, you actually only need to know markdown. The basic markdown syntax is well described [here](https://www.markdownguide.org/basic-syntax/).

## Edit pages
You can edit pages directly on [github](https://github.com/DiedrichsenLab/diedrichsenlab.github.io) by clicking on the edit button on the top right corner of the page.

A better strategy is to clone the repository to your local computer and edit the files there. You can then push the changes to the master branch on github and they will be automatically deployed to the webpage.

## Adding new guide pages
To add a new guide, simply create a new <num><myGuideName>.md file in the folder _guides. Simply start the page with

```
---
layout: page
title: Guide title
---
```

and follow it with your markdown content.



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

## Read more

A good guide to Jekyll is here:

[https://jekyllrb.com/](https://jekyllrb.com/)

And an introduction to liquid syntax here:

[https://shopify.github.io/liquid/basics/introduction/](https://jekyllrb.com/)

