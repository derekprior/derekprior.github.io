---
layout: post
title: "Embracing Promiscuous Gemfiles"
date: 2011-12-13 16:42
summary: By removing version constraints from your Gemfile until you have a
  good reason to add one, the bundle is simple to update and yet still provides
  equivalent version safety.
---

[Bundler](http://gembundler.com) has been around for quite some time now, but I
continue to see what I consider to be be bad advice with regards to specifying
version constraints in your project `Gemfile`. My position is simple: **Do not
constrain gem versions in your Gemfile until you have a good reason to do so**.
I typically encounter two arguments against this position. The first is
misinformed, showing a critical lack of understanding of how bundler works. The
second is a defensible position, but seems to offer no advantage over my
approach.

## Specific Versions For Everything!

Developers in the first camp routinely lock gems down to specific versions
(`'3.1.0'`) They argue that this ensures all developers and all deployment
environments run the same version of the project's dependencies. While this is
true for direct dependencies listed in the `Gemfile`, it is not true for the
dependencies of those dependencies. This position shows a fundamental
misunderstanding of how bundler works. The complete versioning picture is
maintained by bundler in `Gemfile.lock`, which is generated when `bundle
install` is run for the first time. Subsequent runs of `bundle install` install
only the exact versions listed in `Gemfile.lock`. This is why it's critical that
`Gemfile.lock` be checked into source control. It's `Gemfile.lock` -- and not
`Gemfile` -- that provides complete consistency across deployments.

<blockquote>
  <p>This is important: <strong>the Gemfile.lock makes your application a
  single package of both your own code and the third-party code it ran the last
  time you know for sure that everything worked</strong>. Specifying exact
  versions of the third-party code you depend on in your Gemfile would not
  provide the same guarantee, because gems usually declare a range of versions
  for their dependencies.</p>

  <footer>
    <cite>
      <a href="http://gembundler.com/rationale.html">Bundler Rationale</a>
    </cite>
  </footer>
</blockquote>


## Pessimistic Version Constraints for All!

Some developers lean on the use of the [pessimistic version
constraint](http://docs.rubygems.org/read/chapter/16#page74) (`~>`). In fact,
the rubygems web interface encourages this explicitly on every gem page. In the
absence of an existing version that breaks compatibility I find this unnecessary
as well.

The pessimistic constraint is typically used to lock a gem to a specific
major.minor version while allowing the patch level to increase. For instance,
`'~> 3.1.0'` would lock the gem to version 3.1.x. The effectiveness of this
approach depends on gem authors strictly following a
[rational](http://docs.rubygems.org/read/chapter/7) or
[semantic](http://semver.org) versioning policy. Even well-meaning gem
maintainers accidentally introduce breaking changes or performance regressions
in patch number releases. A patch level release may also take new versions
(major, minor, or patch level) of its dependencies which may cause issues with a
different direct dependency in your gemfile.

Regardless of the constraints used in the `Gemfile`, the developer must still
exercise caution when updating the bundle. It's unclear to me what advantage
using pessimistic version constraints by default offers.

## Promiscuous Gemfiles

I opt to leave my gems unconstrained by default. Updating gems in the
application bundle is done with `bundle update <gemname>` and will change
`Gemfile.lock`. This is a conscious operation and is undertaken with care. If
the project's test suite or QA plan fail then it's appropriate to add a version
constraint to the offending gem or update the application to remedy the
incompatibility.

I will leverage a pessimistic version constraint when, for instance, maintaining
a rails 2.3.x project. The `'~> 2.3.14'` constraint would allow the project to
update to newer security fixes while not introducing changes I know to be
breaking from the 3.x releases. I will use an exact version constraint if, for
example, a gem introduces breaking changes in a patch number release. The result
is a less noisy `Gemfile` where all of the version constraints are immediately
obvious and usually documented with appropriate comments.

The introduction of bundler 1.1 (currently at release candidate 3), adds the
command `bundle outdated` which is a boon to this approach. `bundle outdated`
returns a list of gems that would be updated if `bundle update` were run with no
arguments. This allows me to easily see which gems have been updated and
investigate the changes before running `bundle update`. It's important to note
that `bundle outdated` will not show you versions that don't meet the
constraints specified in your gemfile. If you have a gem specified with the
constraint `'~>2.0.0'` and the maintainer releases version 2.1 `bundle outdated`
won't alert you to this. When paired with an unconstrained `Gemfile`, `bundle
outdated` can help developers keep up with changes to dependencies that might
prove useful in their project.

## Conclusion

Unnecessary constraints make updating gems a hassle, particularly as your
project ages or new versions of your dependencies and their dependencies are
released. If you alter a constraint to pick up a new version of a direct
dependency, you may find that `bundle update` fails until you alter additional
constraints. This hoop jumping can be avoided by removing all but the strictly
necessary constraints and employing `bundle update` with care.
