---
layout: post
title: "Gemify Assets for Rails"
date: 2012-01-11 15:47
summary: The asset pipeline allows us to depend on versioned external assets via
  RubyGems, but what happens if the asset you want to use isn't available as a
  Gem? Let's create an asset gem together!
---

The [asset pipeline][ap], introduced with Rails 3.1, makes it simple to include
versioned external assets as application dependencies. Provided those assets are
packaged as Ruby gems, the process is as simple as adding the gem to your
`Gemfile`, running `bundle install` and, in the case of CSS and JavaScript,
adding a require to the proper manifest file.

Many popular CSS frameworks and JavaScript libraries are already available as
gems. Search [RubyGems][rg] to see if the assets you're interested in are
already packaged this way. What if the JavaScript library you use isn't yet
available as a gem? Packaging and publishing asset gems is simple. Here's your
chance.

External assets are made available in Rails via [Rails engines][re]. When the
engine is loaded into your Rails application, the engine's asset paths are added
to your application's load paths. This makes them available for require in your
manifest files. An asset gem is just an absurdly simple engine. As an example,
I'll walk you through the process I recently took for [momentjs-rails][mjs]. The
process is wordy, but it's really pretty simple.

## Create Gem Framework

Bundler makes it simple to create the files and folders necessary for creating a
gem. Run the following command to create and initialize a Git repository along
with several template files for the gem.

```sh
$ bundle gem momentjs-rails
```

## Versioning

In this case, momentjs-rails is a gem packaged version of the [Moment.js][mom]
library. Its version should track the version of JavaScript library it wraps,
which is currently 1.3.0. Open `lib/momentjs-rails/version.rb` and set the
version like so:

```ruby
module Momentjs
  module Rails
    VERSION = "1.3.0"
  end
end
```

In the event that you need to push an update to your gem that does not change
the version of the assets it wraps, I would investigate adding a fourth value to
the version string (`1.3.0.1`).

## Start Your Engine

Bundler created the gem as a standard Ruby module, but we want it to be a Rails
engine. Edit `lib/momentjs-rails.rb` to subclass `Rails::Engine` like so:

```ruby
require "momentjs-rails/version"

module Momentjs
  module Rails
    class Engine < ::Rails::Engine
    end
  end
end
```

Yes, the class is empty on purpose. All we're doing here is declaring the gem as
an engine. This will cause rails to add its directories to the load path when
the gem is required.

## Add The Assets

Download the asset files you want to include in the gem. If availble, I prefer
to use the uncompressed, non-minified versions. They'll actually be readable if
you need to dig into them and you can rely on the asset pipeline to minify them
in production or on precompilation. In the case of the Moment.js library, I
downloaded the release version to my desktop and ran the following commands:

```sh
$ mkdir -p vendor/assets/javascripts
$ cp ~/Downloads/moment.js vendor/assets/javascripts/moment.js
```

I used the `vendor/assets` directory here because these aren't assets I'm
maintaining directly. If they were assets I was responsible for maintaining, I
would put them in `app/assets` though the difference is purely semantic.

## Complete The Gemspec

The `momentjs-rails.gemspec` file contains all of the details that describe the
gem. Open it up and complete any pending `TODO`s left by Bundler. I usually set
the homepage to the GitHub repository for the gem. Additionally, I removed the
`gem.executables` and `gem.test_files` lines, as this gem has neither. I also
prefer to change the `gem.files` setting to avoid shelling out to git. I used
the following pure-ruby implementation instead:

```ruby
gem.files = Dir["{lib,vendor}/**/*"] + ["MIT-LICENSE", "README.md"]
```

Railties needs to be declared as a dependency of the gem. Our gem needs Rails
3.1 or greater, but we'll stop short of 4.0 for now. Use the following setting:

```ruby
gem.add_dependency "railties", "~> 3.1"
```

## Build and Test

Bundler set up some default rake tasks for us that save a few characters when
building. Simply run `rake build` to build the gem.

I haven't included any specs with this gem. Full-featured engines usually have a dummy Rails application in the test directory that loads the engine and can be used to run tests. While this would work to test the asset gem, I haven't set this up yet. Instead, I generally do the following:

```sh
$ rails new momentjs-test
$ cd momentjs-test
$ echo 'gem "momentjs-rails", :path => "/Absolute/Path/To/Gem/Directory"' >> Gemfile
$ bundle install
$ echo '//= require moment' >> app/assets/javascripts/application.js
$ rails server &
$ curl http://localhost:3000/assets/moment.js
$ fg
<ctrl-c>
```

That's quite the little dance, but the `curl` command should return the contents
of the moment.js file if everything is wired up correctly.

## README

I included a very simple readme file with the Gem as its sole documentation. I
brifely described Moment.js, supplied simple usage instructions, and a word
about versioning. See the [momentjs-rails readme][read].

## Push To GitHub and RubyGems

Create a GitHub repository for your project, stage all of your commits, commit,
and push the code to GitHub.

If you've never published a gem on RubyGems before, you'll need to sign up for
an account there. Your account settings will contain an API key that should be
copied to `~/.gem/credentials`. Once that's done, publishing your gem is as
simple as:

```sh
$ rake release
```

This will create a Git tag for this version, and push the built gem to RubyGems.

[ap]: http://guides.rubyonrails.org/asset_pipeline.html
[rg]: https://rubygems.org
[re]: http://railscasts.com/episodes/277-mountable-engines
[mjs]: https://github.com/derekprior/momentjs-rails
[mom]: http://momentjs.com
[read]: https://github.com/derekprior/momentjs-rails/blob/master/README.md
