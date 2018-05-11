---
layout: page
title: Talks
permalink: /talks/
---

You can hear me talk about things on a weekly basis if you subscribe to [The
Bike Shed], a technical discussion podcast that I co-host. I've also given the
following prepared talks and am available to present them at your conference or
to your coworkers. Please [contact me] if you are interested.

[The Bike Shed]: http://bikeshed.fm
[contact me]: /contact

## Cultivating a Code Review Culture

Code reviews are not about catching bugs. Modern code reviews are about
socialization, learning, and teaching.

How can you get the most out of a peer's code review and how can you review code
without being seen as overly critical? Reviewing code and writing easily
reviewed features are skills that will make you a better developer and a better
teammate.

I feel strongly that being a part of a strong code review culture continues to
make me a better developer each day. My goal with this talk is to give each of
you the tools to get more from the code reviews you receive and have more impact
with the reviews you leave for others, yielding your own strong code review
culture.

{% include youtube.html id=PJjmw9TRB7s %}

[railsconf]: https://www.youtube.com/watch?v=PJjmw9TRB7s

## In Relentless Pursuit of REST

The oft-cited benefits of REST tend to revolve around data access via API, but
in this talk we'll look at how a laser-like focus on the language of resources
in your application can lead to better, more maintainable code.

RESTful architecture can narrow the responsibilities of your Rails controllers
and make follow-on refactorings more natural. In this talk, you'll learn to
refactor code to follow RESTful principles and to identify the positive impact
those changes have throughout your application stack.

{% include youtube.html id=HctYHe-YjnE %}

## Up & Down Again: A Migration's Tale

You run `rake db:migrate` and `rake db:schema:load` regularly, but what do they
actually do? How does `rake db:rollback` automatically reverse migrations and
why can't it reverse all of them? How can you teach these tasks new tricks to
support additional database constructs?

We'll answer all of this and more as we explore the world of schema management
in Rails. You will leave this talk with a deep understanding of how Rails
manages schema, a better idea of its pitfalls, and ready to bend it to your
will.
