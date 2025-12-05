---
layout: page
title: Contributing to the WIKI
date: 2023-10-19
category: wiki
description: How can you contribute to this wiki?
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



## Building Local - Install Jekyll
If you want to build and test changes to the WIKI locally, you need an installation of Jekyll 

General instruction:
https://jekyllrb.com/docs/installation


One you have everything installed you can call: 

```
bundle exec jekyll serve
```

and paste the URL into your web-browser. Everytime you save a change to the files in the WIKI, the website will automatically update! 

###  Tips for installing on MacOSX 

First, make sure you have the developer commandline tools installed (gcc, g++, git). 
If not, you do **not** need the full 40GB of Xcode, just follow these instructions to get the command-line tools only: 
[https://www.freecodecamp.org/news/install-xcode-command-line-tools/](https://www.freecodecamp.org/news/install-xcode-command-line-tools/)

On my mac, following the Jekyll instructions above and use
`ruby-install ruby 3.1.3`
Failed, so I installed ruby with brew

`brew install ruby`

And added to .zshrc

```
export PATH="/usr/local/opt/ruby/bin:$PATH"
```

After restarting the terminal, I could install bundler and jekyll with

`gem install jekyll`

and 

`bundle install`


## Read more

A good guide to Jekyll is here:

[https://jekyllrb.com/](https://jekyllrb.com/)

And an introduction to liquid syntax here:

[https://shopify.github.io/liquid/basics/introduction/](https://jekyllrb.com/)

