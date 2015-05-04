---
layout: post
title: "Simple Navigation in Rails"
date: 2011-12-30 15:41
summary: The Simple Navigation gem makes advanced, multi-level navigation
  schemes simple to implement while maintaining a clear separation of concerns.
---

Most web applications call for at least one level of navigational structure.
Many will have two or more levels and require the active node in each level to
have distinct styling via an `active` class. I've seen as many navigation
solutions as I've seen rails projects. Most look similar in that each level of
navigation is represented by a partial. Some track the active node via local
variables passed with the call to `render`, but this approach isn't very DRY.
Some track the active node via instance variables on the controller, but this is
a painful violation of MVC which gets especially tedious to maintain in
multilevel scenarios.

Thankfully, there's a mature, simple, and actively maintained solution just a
gem away. [Simple Navigation][sn] handles just about whatever you can throw at
it. I was surprised, however, by how hard it was to unearth when I was looking
for the solution to my navigational troubles.  It's an elegant solution that
deserves more attention.

With Simple Navigation, the application's entire navigational structure is
contained in a single configuration file. The configuration is specified in a
simple DSL that allows for multiple levels of navigation. You can render the
entire structure, with appropriate sub menus with a call to `render_navigation`
or you can render each level explicitly with calls to `render_navigation :level
=> 2`. There is, of course, support for configuring the exact classes, elements,
and ids used in the HTML.

Active item tracking can be done automatically, which works surprisingly well in
basic cases. Alternatively, you can supply a regular expression or proc to
determine which item should be tracked as active. If your navigation has more
than one level, then the parent of an active item will also be marked as active.

The [project wiki][wiki] has good documentation for most features. There are a
lot of options, but the configuration is still easily understood. For instance,
here's a chunk of the configuration for my current project. The navigation
features a static top bar with any applicable secondary menu rendered as tabs
below. The entire 'Admin' menu is visible only to users who are administrators.

```ruby
SimpleNavigation::Configuration.run do |navigation|
  navigation.autogenerate_item_ids = false
  navigation.selected_class = 'active'

  navigation.items do |primary|
    primary.item :schedule, 'Schedule', schedules_path

    primary.item :admin, 'Admin', admin_root_path, :if => Proc.new { current_user.administrator? } do |admin|
      admin.item :admin_shifts, 'Shifts', admin_shifts_path, :highlights_on => /admin\/?$|admin\/shifts/i
      admin.item :admin_responsibilities, 'Responsibilities', admin_responsibilities_path, :highlights_on => :subpath
      admin.item :admin_locations, 'Locations', admin_locations_path, :highlights_on => :subpath
      admin.item :admin_holiday_schedules, 'Holidays', admin_holiday_schedules_path, :highlights_on => :subpath
      admin.item :admin_users, 'Users', admin_users_path, :highlights_on => :subpath
      admin.dom_class = 'tabs span16'
    end

    primary.item :preferences, 'Preferences', preferences_path
    primary.item :help, 'Help', help_path
    primary.dom_class = 'nav'
  end
end
```

Check out Simple Navigation for use in your Rails projects. It's simple,
powerful, and DRY.

[sn]: https://github.com/andi/simple-navigation
[wiki]: https://github.com/andi/simple-navigation/wiki
