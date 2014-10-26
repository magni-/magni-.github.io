---
layout: post
title:  "Hello World!"
date:   2014-10-26 21:02:01 UTC
categories: jekyll
---

This weekend, I decided to finally join the GitHub Pages with Jekyll party. It's as easy to set up as you've heard. Just follow [GitHub's instructions][github-instructions] and you'll be good to go. The only step that's missing is fairly straightforward: creating a new Jekyll site, _after_ installing Jekyll, but _before_ running `bundle exec jekyll serve`.

It's just one line: `jekyll new .`

If you've followed GitHub's instructions though, you may run into this error:

`Conflict: /Users/paul/mysite exists and is not empty.`

Fortunately, all you need to do is move everything out, re-run the command, and move everything back in.

{% highlight bash %}
mysite$ mkdir ~/tmp
mysite$ mv * ~/tmp
mysite$ jekyll new .
mysite$ mv ~/tmp/* .
mysite$ rmdir ~/tmp
{% endhighlight %}

This is a just a `Hello World!` post, the next should be more technical than a primer on `mkdir` and `mv` :)

[github-instructions]: https://help.github.com/articles/using-jekyll-with-pages/
