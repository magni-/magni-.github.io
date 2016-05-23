---
layout: post
title:  "Making git history great again"
date:   2016-05-20 14:04:03 UTC
categories: git
comments: true
---

My [previous post]({% post_url 2015-08-31-git-patch %}) covered how to write bitesize commits (spoiler: use `git add -p`).

{% highlight nolang %}
$ git add -p foo.rb  # only add bug fix
$ git commit -m "Fix qux bug"
$ git add -p foo.rb  # add remaining spacing fixes
$ git commit -m "Clean up spacing"
{% endhighlight %}

And there you go, a very clean git history! But what if you notice another spacing error after you've already commited?

{% highlight nolang %}
$ git add -p foo.rb  # add forgotten spacing fix
$ git commit -m "More spacing fixes"
{% endhighlight %}

Ugly. Let's try that again.

{% highlight nolang %}
$ git add -p foo.rb  # add forgotten spacing fix
$ git commit --amend --no-edit
{% endhighlight %}

Much better. The `--amend` option adds your changes to the latest commit. The `--no-edit` option means it'll keep the original commit message. Congrats, you've rewritten history!

It's not always so easy though. Let's say that after running all tests, you realize your bugfix broke one obscure test you didn't even know existed. You fix that test to pass, and then what?

{% highlight nolang %}
$ git add -p test_bar.rb  # add fix to old test
$ git commit -m "Fix old test"
{% endhighlight %}

Ugly. There's one more step you need to do.

## git rebase -i

What we want to do is rewind history, skip to before that spacing fix commit, and then amend our test fix changes into the original bugfix commit. `git rebase -i` lets us do that.

The naïve way of doing this would be to stash our changes (rather than commit them), and then opt to _edit_ the bug fix commit:

{% highlight nolang %}
$ git add -p test_bar.rb  # add fix to old test
$ git stash
$ git rebase -i @~1  # will bring up the last two commits
{% endhighlight %}

This will bring the interactive rebase editor:

{% highlight nolang %}
pick abc1234 Fix qux bug
pick 567defg Clean up spacing
{% endhighlight %}

Since we want to change the first commit, we replace that first `pick` with `edit` and save out of the editor. That will start the rebase, pausing after the first commit.

{% highlight nolang %}
$ git stash pop
$ git commit --amend --no-edit
$ git rebase --continue
{% endhighlight %}

That works, but there's a better way of doing it. Rather than dealing with the stash and interrupting the rebase, we can use another tool from the interactive rebase editor: `fixup`. Fixup squashes a commit into the previous one, discarding its commit message. Back to our example:

{% highlight nolang %}
$ git add -p test_bar.rb  # add fix to old test
$ git commit -m "whatever I'm getting squashed anyways"
$ git rebase -i @~2  # we have an extra commit this time around
{% endhighlight %}

Back to the interactive rebase editor:

{% highlight nolang %}
pick abc1234 Fix qux bug
pick 567defg Clean up spacing
pick hij890k whatever I'm getting squashed anyways
{% endhighlight %}

This time, let's move the last commit to be behind the first one, and replace `pick` with `fixup`:

{% highlight nolang %}
pick abc1234 Fix qux bug
fixup hij890k whatever I'm getting squashed anyways
pick 567defg Clean up spacing
{% endhighlight %}

Save out, and the rebase will autorun to completion<sup>[_[1]_](#footnote-1)</sup>. This gives us the same result as the stash strategy, but it still feels like a little too much work.

We can shave off part of the work by using `git rebase`'s `--autosquash` flag. That flag looks for specially formatted commits in order to prepare the rebase for you. Back to our example:

{% highlight nolang %}
$ git add -p test_bar.rb  # add fix to old test
$ git commit -m "fixup! Fix qux bug"
$ git rebase -i @~2
{% endhighlight %}

And now the editor opens up with our commits in the desired order and with the desired actions:

{% highlight nolang %}
pick abc1234 Fix qux bug
fixup hij890k whatever I'm getting squashed anyways
pick 567defg Clean up spacing
{% endhighlight %}

You can instantly save out, and the rebase will run. This saves us work in the interactive rebase editor, but now we need to worry about properly typing out our throwaway commit's message. Surely there's a better way?

The final piece: `git commit`'s `--fixup` flag. Rather than manually adding a message, we can use this flag and pass a commit reference or SHA, and `git` will autogenerate the correct commit message for us:

{% highlight nolang %}
$ git add -p test_bar.rb  # add fix to old test
$ git commit --fixup=@~1
$ git rebase -i @~2
{% endhighlight %}

This will give us the same text in the interactive rebase editor as above. Once again, all you need to do is save out.

<a name="footnote-1">1.</a> Unless you have a rebase conflict somewhere.