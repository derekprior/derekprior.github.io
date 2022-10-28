---
layout: post
title: Testing Ruby Gems with GitHub Actions
summary: Let's walk through how you can use GitHub Actions to test a Ruby gem,
  or any other Library, against an array of dependencies.
date: 2020-04-10 22:00
---

[Scenic] is a Ruby gem that adds methods to `ActiveRecord::Migration` to create
and manage versioned database views in Rails. We've been using Travis CI's build
matrix feature to test Scenic across several supported combinations of Ruby and
Rails since Scenic's inception. It's been quite a while since Travis was my
preferred CI vendor, but the simple build matrix setup kept us on Travis.

Then came GitHub Actions. Actions is a general purpose platform to automate all
software workflows. I've been interested in converting Scenic's test suite to
run on Actions for quite some time, but the available CI workflow templates for
Ruby applications are geared towards testing applications. I knew I'd have some
additional work to do.

In this post, we'll walk through Scenic's non-trivial workflow for running tests
on GitHub Actions.

## Our Workflow

We specified our workflow in `.github/workflows/ci.yml` ([full
source][workflow]). In this post we'll break down the workflow to discuss what's
happening in each part. The [workflow syntax documentation][workflow docs] is
quite good if you want more detail on any particular bit.

```yaml
name: CI

on:
  push:
    branches: master
  pull_request:
    branches: "*"
```

The workflow `name` is what we'll see in the Actions tab on our repository and
alongside the builds on pull requests.

In `on`, we specify the events we want this workflow to respond to. In our case,
we want the workflow to run for any push to `master` or on any pull request
targeting any branch.

```yaml
jobs:
  build:
    strategy:
      fail-fast: false
      matrix:
        ruby: ["2.7","2.4"]
        rails: ["5.2", "6.0", "master"]
        exclude:
          - ruby: "2.4"
            rails: "6.0"
          - ruby: "2.4"
            rails: "master"
        include:
          - rails: "master"
            continue-on-error: true
```

Workflows are made up of one or more `jobs`. We elected to call our job `build`.
This job specifies a `strategy` that allows us to set up a matrix of job
configurations.

When using the matrix strategy, all running jobs in the matrix will be canceled
as soon as the first configuration fails unless we explicitly disable
`fail-fast` as we've done here. We want all jobs to run to completion so we can
know exactly which configurations failed and which succeeded.

Each configuration varies its  versions of Ruby and Rails. While Scenic supports
many more versions of Ruby and Rails than those listed, we believe explicitly
testing the edges of our matrix gives us adequate coverage and allows us to save
build time and CO2 emissions.

It's important to note that there's nothing special about the `ruby` and `rails`
keys here. They are essentially variables that will be defined as strings for
each configration. I could have called them `ruby-version` and `rails-version`
and you can specify whatever name and number of configuration variables you
want.

The `exclude` configuration allows us to explitly remove configuration
combinations from the resulting matrix. In our case we do not want to test
against Rails 6 or edge Rails on Ruby 2.4 because those Rails versions do not
run on Ruby 2.4.

You might expect that `include` functions as the opposite of `exclude` and
allows us to add configurations not otherwise present in our matrix. It's an
understandable assumption, but it turns out to be incorrect and attempts to use
it in this manner will fail workflow validation. Instead `include` lets us
specify ("include") additional options for configurations that
_are_ in the matrix. In our case, we use `include` to set `continue-on-error` to
`true` whenever a configuration includes Rails master.

{% raw %}
```yaml
    name: Ruby ${{ matrix.ruby }}, Rails ${{ matrix.rails }}
    runs-on: ubuntu-latest
```
{% endraw %}

We customize the name of the job as displayed on GitHub by setting `name`. In
this case we do so referencing the configuration options we set in our matrix,
which will result in jobs named "Ruby 2.7, Rails 5.2", for example. The name of
each job in the Matrix will be visible in our PRs, so we'll know at a glance
which configurations passed and which failed.

The `runs-on` configuration is rather unremarkable for us as we only need to
test on one platform.

```yaml
    services:
      postgres:
        image: postgres
        env:
          POSTGRES_USER: "postgres"
          POSTGRES_PASSWORD: "postgres"
        ports:
          - 5432:5432
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
```

Scenic requires Postgres to run its tests. The `services` configuration allows
us to add a Postgres container to our configuration and have Actions manage its
lifecycle. I have a very limited understanding of containers which makes this
the part of the configuration I understand the least, but it does the trick.

The `options` configuration ensures a health check succeeds before the workflow
continues. This prevents potentially confusing errors surfacing later in the
workflow run should Postgres fail to start cleanly.

{% raw %}
```yaml
    env:
      RAILS_VERSION: ${{ matrix.rails }}
      POSTGRES_USER: "postgres"
      POSTGRES_PASSWORD: "postgres"
```
{% endraw %}

`env` lets us specify environment variables that are available to all the steps
in our job. It's also possible to specify root-level or step-level `env`. We use
environment variables to set the version of Rails in our `Gemfile` and to
optionally set database configuration parameters in our app. More on this later.

```yaml
    steps:
```

This is where we specify each step we want to take in our workflow. Steps can
execute other named actions or run custom commands. We do a bit of both in our
workflow. The `name` key is optional, but I prefer to have one for each step to
serve as documentation for the reader and to make the resulting workflow logs
clearer. If any step returns a failure status, the job will halt and be marked
as a failure.

Let's take it step-by-step from here.

```yaml
      - name: Checkout
        uses: actions/checkout@v2
```

Here you can see our first example of referencing an action in our workflow.
This particular action clones our repository to our workspace.
`actions/checkout` refers to the repository on GitHub where this action is
defined and `@v2` refers specifically to the tag `v2` in that repository.

{% raw %}
```yaml
      - name: Install Ruby ${{ matrix.ruby }}
        uses: ruby/setup-ruby@v1.14.1
        with:
          ruby-version: ${{ matrix.ruby }}
```
{% endraw %}

We're using another named action to install the version of Ruby the current
configuration uses.

```yaml
      - name: Install dependent libraries
        run: sudo apt-get install libpq-dev
```

In order to successfully install the `pg` gem in later steps, we need to have
the `libpq-dev` package installed. This is our first example of a step that does
not reference another action, but instead directly `run`s a command.

```yaml
      - name: Generate lockfile
        run: bundle lock
```

Scenic, because it's a library as opposed to an application, does not commit
it's `Gemfile.lock`. Explaining why that is is beyond the scope of this article
but it has consequences for how we configure the workflow. This step, together
with the matrix setup above, are likely the biggest differences between testing
a library with Actions and testing an application with Actions.

The `bundle lock` command does all the work of figuring out which dependency
versions satisfy your `Gemfile` and generates the `Gemfile.lock` with this
information. In that way, it's very similar to the more familiar `bundle
install`, except that `bundle lock` stops short of actually installing
dependencies.

We want the lockfile to be present so we can use it in our next step, which
might save us the time of the dependency install step.

{% raw %}
```yaml
      - name: Cache dependencies
        uses: actions/cache@v1
        with:
          path: vendor/bundle
          key: bundle-${{ hashFiles('Gemfile.lock') }}
```
{% endraw %}

We're using the cache action to cache the `vendor/bundle` path, which is where
we will ultimately be installing our dependencies. We hash the contents of
`Gemfile.lock`, which we generated in the previous step, to use as part of our
cache key. If `Gemfile.lock` is unchanged between runs, the cache action will
restore the cache to `vendor/bundle`.

The cache action also automatically adds a step to the end of our workflow which
will cache the contents of `vendor/bundle` using our key for future workflow
runs.

In many ways, this is equivalent to Travis' cache configuration except that the
cache action is much more flexible. In order to get this step correct, I had to
better understand how dependency caching works -- which paths were cached, what
was used as the key -- which lead me to discover that caching was not working as
we intended on Travis. A simple configuration is of no use if it doesn't work.

```yaml
      - name: Set up Scenic
        run: bin/setup
```

Like most of my projects, Scenic features a [`bin/setup`][setup] script for
repeatable development configuration. I like to run these scripts as part of the
CI setup so they are exercised frequently. There are some things happening in
the setup script that are of interest to our workflow, so let's take a look at
it:

```sh
#!/bin/sh

set -e

# CI-specific setup
if [ -n "$GITHUB_ACTIONS" ]; then
  bundle config path vendor/bundle
  bundle config jobs 4
  bundle config retry 3
  git config --global user.name 'GitHub Actions'
  git config --global user.email 'github-actions@example.com'
fi

gem install bundler --conservative
bundle check || bundle install

bundle exec rake dummy:db:drop
bundle exec rake dummy:db:create
```

The first thing that jumps out at you is likely the conditional block based on
the `$GITHUB_ACTIONS` environment variable. This variable is set by the actions
platform, allowing us to conditionally perform some action when executing there.
In our case, we have some additional tasks when the setup script is running on
CI:

* Configure bundler as appropriate for CI. Most importantly this includes
  setting our install path as `vendor/bundle` as used by our caching step above.
* Configure git with a name and an email. The scenic acceptance test suite
  creates commits which require these git configurations to be set.

Finally, the last two lines of the script reinitialize the database used by the
test suite. There is a [`database.yml`][database] file which conditionally uses
the `POSTGRES_USER` and `POSTGRES_PASSWORD` environment variables we set earlier
in our workflow:

```yaml
development: &default
  adapter: postgresql
  database: dummy_development
  encoding: unicode
  host: localhost
  pool: 5
  <% if ENV.fetch("GITHUB_ACTIONS", false) %>
  username: <%= ENV.fetch("POSTGRES_USER") %>
  password: <%= ENV.fetch("POSTGRES_PASSWORD") %>
  <% end %>

test:
  <<: *default
  database: dummy_test
```

Okay, let's get back to our Workflow file. We're finally ready to run our tests!

{% raw %}
```yaml
      - name: Run fast tests
        run: bundle exec rake spec
        continue-on-error: ${{ matrix.continue-on-error }}

      - name: Run acceptance tests
        run: bundle exec rake spec:acceptance
        continue-on-error: ${{ matrix.continue-on-error }}
```
{% endraw %}

Scenic has two different test suites. These can be run together with `rake`, but
I prefer to run them as separate steps. The "fast tests" take less than three
seconds to complete and if any of them fail, we don't want to run the much
slower acceptance tests. Those take about 40 seconds.

You might have noticed the `continue-on-error` key for both of these steps.
The value here is dependent on our matrix. As discussed earlier, it will be
`true` for any configuration that features Rails master. When set,
`continue-on-error` prevents the step from failing. This has the effect of
allowing our builds against Rails master to run without marking pull requests
themselves as failing if they only fail tests against edge Rails.

## The Results

The workflow configuration is longer and more involved than the
mostly-equivalent Travis configuration. I don't find the Workflow configuration
to be too intimidating but it gets "a little weird" as it relates to lockfile
generation and bundle caching. However, this weirdness served to highlight that
the simpler caching configuration offered by Travis was insufficient for our use
case.

When it comes to performance, Actions is a clear winner. The accurate caching
certainly helps, but setting that aside, I've found that jobs start running
faster than they did on Travis. Additionally, Actions has much higher
concurrency limits, allowing 20 concurrent jobs on the free plan. This means
Scenic's full matrix executes concurrently on actions, while on Travis we are
frequently limited to two concurrent builds.

On the user experience front, Actions is once again the clear winner due to each
job's representation directly in the UI. I do wish that there was a way to
visually indicate an allowed failure. As it is, the overview UI provided on PRs
pretends that edge Rails builds are passing due to our use of
`continue-on-error`. You don't discover that they are actually failing until you
view more details and see the job annotations.

Setting up a non-trivial workflow taught me a lot about how I can achieve
aritrary things with Actions. I've already put some of this to use in my day
job, and have a list of other workflows I'd like to codify and automate with
Actions as well.

If this walkthrough was helpful to you, please let me know on [Twitter].

[Scenic]: https://github.com/scenic-views/scenic
[workflow]: https://github.com/scenic-views/scenic/blob/4d52ba6ce3ff51dfd103f478b25d7a565b7d358b/.github/workflows/ci.yml
[workflow docs]: https://help.github.com/en/actions/reference/workflow-syntax-for-github-actions
[setup]: https://github.com/scenic-views/scenic/blob/4d52ba6ce3ff51dfd103f478b25d7a565b7d358b/bin/setup
[database]: https://github.com/scenic-views/scenic/blob/4d52ba6ce3ff51dfd103f478b25d7a565b7d358b/spec/dummy/config/database.yml
[Twitter]: https://twitter.com/derekprior
