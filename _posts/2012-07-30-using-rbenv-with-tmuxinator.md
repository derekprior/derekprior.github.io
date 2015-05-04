---
layout: post
title: "Using rbenv with tmuxinator"
date: 2012-07-30 11:13
summary: Tmuxinator allows for easy configuration of tmux sessions, but it has
  not yet been updated for use with rbenv. A simple shell function can serve as
  to bridge the divide.
---

[Tmuxinator][mux] allows for easy configuration of [tmux][tmux] sessions. Its a
handy gem, but hasn't received any major updates in quite some time. While it
has support for specifying a project-level [rvm][rvm] version, it has not yet
accepted either [pull][pr1] [request][pr2] that would add proper [rbenv][rbenv]
support. Not wanting to install a fork from GitHub, I found I could easily use
the existing rvm support if I just shimmed calls to rvm into the equivalent call
in rbenv.

Tmuxinator will call `rvm use` and pass the version identifier
set in the `rvm` key of the configuration file. All I had to do was translate
this into the equivalent call to `rbenv shell`. I added the following function
to my zsh configuration:

```sh
function rvm () {
  if [[ $1 == 'use' ]]; then
    rbenv shell $2
  fi
}
```

I'm a total shell hack and recognize that this isn't very robust, but it gets
this sepecific job done. If all you need is for the tmuxinator-initiated tmux
session to respect the `rbenv local` setting for the initial directory, then
[Ian Yang's solution][ian] may work great for you. In my case, I wanted all
initialized windows to use the same version of ruby, regardless of whether
they had a local ruby version set. As an added bonus, expanding this function
to translate other calls may help other rvm-supporting tools support rbenv.

[mux]: https://github.com/aziz/tmuxinator/
[tmux]: http://tmux.sourceforge.net
[rvm]: http://rvm.beginrescueend.com/
[rbenv]: https://github.com/sstephenson/rbenv/
[pr1]: https://github.com/aziz/tmuxinator/pull/45
[pr2]: https://github.com/aziz/tmuxinator/pull/36
[ian]: http://log.iany.me/post/16816192156/fix-rbenv-in-tmuxinator
