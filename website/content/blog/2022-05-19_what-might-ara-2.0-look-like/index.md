---
author: "David Moreau Simard"
categories:
  - development
tags:
  - ansible
date: 2022-05-19
title: "What might ara 2.0 look like ?"
slug: what-might-ara-2.0-look-like
type: post
---

I've been wanting to write about this for a while and I'm interested in what you think. Head to this [GitHub issue](https://github.com/ansible-community/ara/issues/388) for discussing this post.

Broadly speaking, in [semantic versioning](https://semver.org/), you bump a major version to signal users that they should expect backwards incompatible changes.

## About the jump from 0.x to 1.x

ara has loosely adhered to semantic versioning and so far has only ever needed to do this once:

- The first release of the project was based on [flask](https://flask.palletsprojects.com) with version [0.1 on 2016-05-08](https://github.com/ansible-community/ara/releases/tag/0.1), all the way up to [0.16.7 on 2020-04-14](https://github.com/ansible-community/ara/releases/tag/0.16.7).
- Then, there was [1.0.0 on 2019-06-04](https://github.com/ansible-community/ara/releases/tag/1.0.0) based on [django](https://www.djangoproject.com/) & [django-rest-framework](https://www.django-rest-framework.org/) with the latest version being [1.5.8 from 2022-03-31](https://github.com/ansible-community/ara/releases/tag/1.5.8).

There would have been a version 1.0 based on flask which added an API with [flask-restful](https://flask-restful.readthedocs.io/en/latest/) but it was abandoned in favor of an implementation with django and django-rest-framework instead.

I was able to make things work with flask-restful and made good progress ([the tag still exists, if you're curious](https://github.com/ansible-community/ara/releases/tag/old-flask-1.0)) but there were issues while so many things came "for free" with django and DRF.

So, anyway, 1.0.0 marked the release of a rewrite that I felt was worthy of a major version bump.

In practice, here's what some of the backward incompatibilities looked like:

- Largely incompatible database models (flask to django) with no easy or simple migration path
- Installing ara was a bit different:
  - in 0.x, you got everything with ``pip install ara`` (ansible plugins, flask web app) but it meant pulling a lot of dependencies
  - in 1.x, ``pip install ara`` contains /only/ the ansible plugins and the (new) API server dependencies are under an extra instead: ``pip install ara[server]``
- Because there was now an actual API, the callback plugin sent results to an API server instead of making raw queries with flask-sqlalchemy
- The web interface was very different:
  - from css with [patternfly](https://www.patternfly.org/) to [bootstrap](https://getbootstrap.com/)
  - from flask templating to django templating
  - it didn't look and feel the same (it still doesn't and probably never will)
  - it wasn't particularly pretty (it could still be better even if there's been a lot of improvements since then -- we could use help for the frontend, \**wink*\*)

But, in general, ara had the same purpose and worked the same way -- you installed ara, enabled the callback plugin and there you go: simple reporting for your ansible-playbook commands without needing to change your workflow.

## Why a major version bump ?

Now, what might an ara 2.0 look like ?

What features might justify changes significant enough to warrant a version bump ?

Nothing as groundbreaking as 1.0, at least not for the time being.

I will spare you the details for now but I have learned a lot by working on ara and there are some mistakes that I would like to fix.

We can't address them all at once but first, I would like to use 2.0 as an opportunity to do significant changes to the [database model](https://github.com/ansible-community/ara/blob/master/ara/api/models.py).

In a nutshell, the [current implementation](https://ara.readthedocs.io/en/latest/api-documentation.html#relationship-between-objects) helps keep the database size in check by compressing files and text, making the data binary instead of text which makes it harder (and more expensive) to search into.

Text (like YAML files or ansible-playbook output) compresses very well so don't get me wrong -- it works and has accomplished it's intended purpose well: last month, the demo database weighed in at only [1.7GB with records from over 3000 playbooks and 700 000 results](https://twitter.com/RecordsAnsible/status/1516949433673469952).

But, with the power of hindsight and experience, some fields should have not been compressed in the first place.

As a user of ara myself, I have come across use cases where I would have liked to be able to search for things inside particular fields -- for example:

- searching ansible-playbook CLI arguments for playbooks which have been called against a specific inventory
- searching for hosts based on the values of their host facts
- searching for tasks based on their tags
- searching for results based on whether the content contains a string
- searching for records (from [ara_record](https://ara.readthedocs.io/en/latest/ansible-plugins-and-use-cases.html#ara-record-arbitrary-key-values-in-playbook-reports)) based on their values

These kinds of features are currently hindered by the current database model but it wouldn't be too hard to provide them if we allowed ourselves to potentially break backwards compatibility.

I believe this would be possible by dropping compression for many fields and leverage the "new" django [JSONField](https://github.com/ansible-community/ara/issues/153) instead because it turns out that ansible returns a lot of JSON. It would mean these fields would now be easily searchable at the expense of increased disk usage because they would no longer be compressed.

I'm not sure if I would remove compression on playbook and role files -- I can understand the need for searching through files but I think there are tools better than ara to do that.

I have not experimented with JSONFields and so I don't know the extent of the changes necessary yet.
Providing a SQL migration that decompresses the current content, alters the field's type and re-inserts it as a native JSONField might be possible and I guess there would be changes to the API.

## In conclusion

This is already long enough but I mean to say that maintaining backwards compatibility has tradeoffs and require maintenance.
It isn't always sustainable or even possible depending on the nature of the changes.

The reality is that ara is a simple, modest and humble project with an author that has limited time and skills so things might not be backwards compatible forever -- sorry about that !

So, what do you think ? Would you use these newly-enabled features ? Is there anything else you'd like added, improved or fixed that could require a major version bump ?

Head on to this [GitHub issue](https://github.com/ansible-community/ara/issues/388) to brainstorm or share your comments and feedback.

Thank you !
