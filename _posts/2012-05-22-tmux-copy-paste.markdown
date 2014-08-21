---
layout: post
title: Copy and paste in Tmux on OSX
class: tmux-copy-paste
description: How to move your Tmux copy buffer to the Mac OSX clipboard
---

After a month of using [Tmux](http://tmux.sourceforge.net/) I can safely say that
I really love how it organizes my workflow. If you haven't used Tmux before, I
highly recommend [Brian Hogan's book](http://pragprog.com/book/bhtmux/tmux). I
have only read the first few chapters, and that was enough for me to get going
and be more productive in Tmux and terminal VIM than I ever was with MacVim. 

That all said, it didn't take long before I ran into a potentially showstopping issue
with Tmux on Mac OSX: copying text within Tmux does not copy it to the OSX 
clipboard.  Luckily, smart people have toiled over this problem and come up with 
a good solution. Here is what I did:

### Install [reattach-to-user-namespace](https://github.com/ChrisJohnsen/tmux-MacOSX-pasteboard/) through [homebrew](http://mxcl.github.com/homebrew/)

{% highlight bash %}
 brew install reattach-to-user-namespace
 
{% endhighlight %}

### Open tmux.conf and paste the following lines

{% highlight bash %}
 set-option -g default-command "reattach-to-user-namespace -l zsh"
  bind y run "tmux save-buffer - | reattach-to-user-namespace pbcopy"
  bind p run "tmux paste-buffer"
 
{% endhighlight %}

Now when you select text with your cursor in Tmux, if you press `prefix + y`
afterwards it will copy the text to your OSX clipboard. `prefix + p`
  will paste from your Tmux buffer, and `⌘ + v` will paste from
your OSX clipboard. A perfect solution would copy the text to the OSX
clipboard without requiring any Tmux commands to be executed by the
user, but I have yet to find a way to do this without a [constantly
polling shell
script](http://robots.thoughtbot.com/post/19398560514/how-to-copy-and-paste-with-tmux-on-mac-os-x).
