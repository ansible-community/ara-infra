---
author: "David Moreau Simard"
categories:
  - development
  - changelog
tags:
  - ansible
  - openstack
date: 2017-02-13
title: Announcing the release of 0.11
slug: announcing-the-release-of-0.11
type: post
---

We're on the road to version 1.0.0 and we're getting closer: introducing the release of version 0.11!

[Four new contributors](https://github.com/openstack/ara/graphs/contributors) (!), [55 commits](https://github.com/openstack/ara/compare/0.10.0...0.11.0) since 0.10 and 112 files changed for a total of 2,247 additions and 939 deletions.

New features, more stability, better documentation and better test coverage.

## The changelog since 0.10.5

- New feature: ARA UI and Ansible version (ARA UI is running with) are now shown at the top right
- New feature: The Ansible version a playbook was run is now stored and displayed in the playbook reports
- New feature: New command: "ara generate junit": generates a junit xml stream of all task results
- New feature: ara_record now supports two new types: "list" and "dict", each rendered appropriately in the UI
- UI: Add ARA logo and favicon
- UI: Left navigation bar was removed (top navigation bar will be further improved in future versions)
- Bugfix: CLI commands could sometimes fail when trying to format as JSON or YAML
- Bugfix: Database and logs now properly default to ARA_DIR if ARA_DIR is changed
- Bugfix: When using non-ascii characters (ex: äëö) in playbook files, web application or static generation could fail
- Bugfix: Trying to use ara_record to record non strings (ex: lists or dicts) could fail
- Bugfix: Ansible config: 'tmppath' is now a 'type_value' instead of a boolean
- Deprecation: The "ara generate" command was deprecated and moved to "ara generate html"
- Deprecation: The deprecated callback location, ara/callback has been removed. Use ara/plugins/callbacks.
- Misc: Various unit and integration testing coverage improvements and optimization
- Misc: Slowly started working on full python 3 compatibility
- Misc: ARA now has a logo

### ARA now has a logo !

Thanks [Jason Rist](https://twitter.com/knowncitizen) for the contribution, really appreciate it !

[With the icon](https://github.com/openstack/ara/blob/master/doc/source/_static/ara-with-icon.png):
![icon](ara-with-icon.png)

[Without the icon](https://github.com/openstack/ara/blob/master/doc/source/_static/ara.png):
![full](ara.png)

## Taking the newest version of ARA out for a spin

Want to give this new version a try ? It's out on pypi!

Install [dependencies and ARA](https://ara.readthedocs.io/en/latest/installation.html), configure the [Ansible callback location](https://ara.readthedocs.io/en/latest/configuration.html#ansible) and ansible-playbook your stuff !

Once ARA has recorded your playbook, you'll be able to fire off and browse the [embedded server](https://ara.readthedocs.io/en/latest/usage.html#browsing-the-web-interface) or generate a [static version](https://ara.readthedocs.io/en/latest/usage.html#generating-a-static-html-version-of-the-web-application) of the report.

## The road ahead: version 1.0

What is coming in version 1.0 ? Let me ask you this question: what would *you* like in 1.0 ?
The development of ARA has mostly been driven by it's user's needs and I'm really excited with what we already have.

I'd like to finish a few things before releasing 1.0... let's take a sneak peek.

### New web user interface

I've been working slowly but surely on a complete UI refactor, you can look at an early prototype preview here:

{{< youtube h3vY87_EWHw >}}

Some ideas and concepts have evolved since then but the general idea is to try and display more information in less pages, while not going overboard and have your browser throw up due to the weight of the pages.

Some ARA users are running playbooks involving hundreds of hosts or thousands of tasks and it makes the static generation very slow, large and heavy.
While I don't think I'll be able to make the static generation work well at any kind of scale, I think we can make this better.

There will have to be a certain point in terms of scale where users will be encouraged to leverage the dynamic web application instead.

### Python 3 support

ARA isn't gating against python3 right now and is actually failing unit tests when running python3.

As Ansible is working towards python3 support, ARA needs to be there too.

### More complex use case support (stability/maturity)

There are some cases where it's unclear if ARA works well or works at all.
This is probably a matter of stability and maturity.

For example, ARA currently might not behave well when running concurrent ansible-playbook runs from the same node or if a remote database server happens to be on vacation.

More complex use case support might also mean providing users documentation on how to best leverage all the data that ARA records and provides: elasticsearch implementation, junit reports and so on.

If ARA is useful to you, I'd be happy to learn about your use case. Get in touch and let's chat.

### Implement support for ad-hoc ansible run logging

ARA will by default record anything and everything related to ansible-playbook runs.
It also needs to support ad-hoc ansible commands as well. I want this before tagging 1.0.

### Other features

There's some other features I'd like to see make the cut for version 1.0:

- [Fully featured Ansible role](https://github.com/openstack/ansible-role-ara) for ARA
- Store variables and extra variables
- Provide some level of support for data on a role basis (filter tasks by role, metrics, duration, etc.)
- Support generating a html or junit report for a specific playbook (rather than the whole thing)
- Packaging for Debian/Ubuntu and Fedora/CentOS/RHEL

A stretch goal would be to re-write ARA to be properly split between client, server, UI and API -- however I'm okay to let that slip for 2.0!

What else would you like to see in ARA ? Let me know in the comments, on IRC in #ara on freenode or on [twitter](https://twitter.com/dmsimard)!

