---
layout: post
title: "Using PeepOpen With RubyMine"
date: 2011-08-11 21:27
summary: Replace RubyMine's keyboard-based file navigation with PeepOpen.
---

I've been doing a lot of development in
[MacVim](https://github.com/carlhuda/janus) of late, but there are times when I
long for the warm embrace of an IDE. For many reasons that I won't detail here,
I find  [RubyMine](http://www.jetbrains.com/ruby/) to be the best Ruby IDE out
there. While its keyboard-based file navigation is functional, I much prefer the
interface provided by [PeepOpen](http://peepcode.com/products/peepopen) that I
had grown acustomed to when using MacVim and TextMate.

Fortunately, with a little work we can use the External Tools configuration in
RubyMine to invoke PeepOpen.

1. With RubyMine closed, navigate to your /Applications folder and rename the
   RubyMine app bundle (i.e. RubyMine 3.2.3) to RubyMine. I tried numerous
   escape sequences when configuring the external tool to avoid this, but none
   of them worked.
2. Launch RubyMine, open preferences, and navigate to the "External Tools"
   section under IDE Settings. Add a new external tool. Name it, group it, and
   describe it however you wish. Make sure you turn off the console option. Set
   the program to `open` and the parameters to
   `peepopen://$ProjectFileDir$?editor=RubyMine`. My configuration looks like
   [this][config]. Apply these changes to save the new command.
3. Go to the KeyMap settings in RubyMine and search for the command you just
   created. Add a keyboard shortcut of your choosing. I used command-t, just as
   I had in MacVim and TextMate.
4. Be sure to apply all changes before exiting the preferences. Load up a
   project and give it a shot!

[config]: /images/peepopen_rubymine.png

## Notes ##

* The `editor=` line in the parameter tells PeepOpen which application it should
  send the file to. The space in the RubyMine app bundle name was causing this
  to fail. I tried numerous ways of escaping this before giving up and renaming
  the bundle. I've [always been annoyed][version] by it, anyway.
* I had some issues with the `$ProjectFileDir$` macro in my project.
  `$Projectpath$` and `$SourcePath$` were also [incorrectly mapped][mapping].
  I fixed this by adding/removing some random items under the Project Structure
  settings, but it's not clear why this was necessary.
* You probably want to open your PeepOpen preferences and set it to exclude the
  `.idea` directory by adding that path to "Directories to Ignore".

[version]: http://youtrack.jetbrains.net/issue/RUBY-8671?projectKey=RUBY
[mapping]: http://devnet.jetbrains.net/thread/311347;jsessionid=E7D0E5AB18FE559CEF75DB75A5761214?tstart=0

