---
layout: post
title: "Upgrading Vim on OS X"
date: 2011-12-22 10:52
summary: The version of Vim that ships with Mac OS X is outdated. There are
  several rather simple options for updating Vim.
---

Mac OS X ships with a console version of Vim, but it is outdated and it was not
compiled with Ruby or Python support.  [Command-T][cmdt], for one, requires Vim
to be compiled with Ruby support and thus will not work with the version of Vim
shipped with the operating system.  You may also have configuration settings in
your `.vimrc` that are incompatible with Vim 7.2. Thankfully, there are a few
rather simple options for updating Vim.

[cmdt]: https://wincent.com/products/command-t

## Simplest: Homebrew built MacVim

[Homebrew][homebrew] is my favorite OS X package manager. If you're not familiar
with Homebrew, you should check it out. You can use Homebrew to install MacVim,
which includes both a GUI and console version of Vim 7.3. Passing a simple flag
to `brew install` will instruct homebrew to setup the necessary symlinks to
replace the system console Vim with the version provided by MacVim.

[homebrew]: http://mxcl.github.com/homebrew/

```sh
$ brew install macvim --override-system-vim

# If you need the app bundle linked in /Applications...
$ brew linkapps
```

__Note__: Please see the caveat below concerning building Vim

## Simple: Homebrew built Vim

If you don't want the version of Vim shipped with MacVim, you can build Vim
directly from source, also using Homebrew. <strike>Homebrew does not include a formula
for Vim in its official repository as that would be counter to its (occasionally
broken) [rule](https://github.com/mxcl/homebrew/wiki/Acceptable-Formula) to not
provide system duplicates. There's a
[formula](https://raw.github.com/Homebrew/homebrew-dupes/master/vim.rb) in the
[homebrew-dupes](https://github.com/Homebrew/homebrew-dupes) repository that
will get the job done in just a few short steps, however.</strike>

**Update**: The Homebrew maintainers have decided to break their rule against
system duplicates in this case. The Vim formula can now be found in the main
repository. The instructions below have been updated and this is now the
simplest way to update console Vim, but I still like having MacVim as well.

```sh
# mercurial required - install if you don't already have it.
$ brew install mercurial

# install Vim
$ brew install vim

# if /usr/bin is before /usr/local/bin in your $PATH,
# hide the system Vim so the new version is found first
$ sudo mv /usr/bin/vim /usr/bin/vim72

# should return /usr/local/bin/vim
$ which vim
```

__Note__: Please see the caveat below concerning building Vim

## Without Homebrew

If you're interested in Vim, I'm guessing you're also fully capable of using a
package manager, but if you want to do this quickly and without involving
Homebrew and without running afoul of the caveat below, you can download and
install the [MacVim](http:code.google.com/p/macvim) app bundle  and setup an
alias to point to its version of Vim. Assuming you have MacVim installed to
`/Applications/`, you can add the folowing to `~/.bash_profile` or other
appropriate configuration file.

```sh
$ alias vim="/Applications/MacVim.app/Contents/MacOS/Vim"
```

## Caveat: Locally Built Vim

If you build Vim locally via Homebrew or any other method and you use
[RVM](http://rvm.beginrescueend.com) or
[rbenv](https://github.com/sstephenson/rbenv) to manage multiple rubies, you
should be sure to switch to the system installed Ruby when building Vim and any
compiled plugins (such as Command-T). If you build Vim against one Ruby and
Command-T against another, the penalty will be a SegFault when launching Vim. In
that case, rebuild both against a consistent Ruby.
