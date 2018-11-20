---
author: "David Moreau Simard"
categories:
  - development
  - changelog
tags:
  - ansible
  - openstack
date: 2016-06-07
title: "One month and 200 commits later"
slug: one-month-and-200-commits-later
type: post
---

On May 21st, I wrote a blog post about [ARA, an idea to store, browse and troubleshoot Ansible playbook runs](https://dmsimard.com/2016/05/21/ara-an-idea-to-store-browse-and-troubleshoot-ansible-playbook-runs/).
Let's rewind a bit further back in time.

On May 6th, I got tired of trying to make our [human_log](https://github.com/rdo-infra/weirdo/blob/master/playbooks/library/human_log.py) callback write user friendly HTML files.
I simply wasn't happy with my [attempts](https://review.gerrithub.io/#/q/topic:human_log_html)...

I'm a big fan of the UNIX philosophy: Do one thing and do it well.
Trying to hack HTML writing into human_log didn't feel like that at all.

I got frustrated and took a completely different direction.
What if, instead of writing and appending to a html file throughout a playbook run, I'd write to a database and make a dynamic, database-driven interface.
This would allow to organize playbook run data and also provide supported for aggregated run visualization.

Not a lot of sleep was had that weekend. I [created a repository](https://github.com/dmsimard/ara/commit/ad09488bc291e6006f79110f903be962ab0d0a39) on friday and set out to have a prototype out by the following monday,

And what a wild ride it's been since then.
I first shared the idea with an [alpha prototype preview](https://www.youtube.com/watch?v=K3jTqgm2YuY).
An awesome collaborator, [Lars Kellog-Stedman](http://blog.oddbit.com/), started contributing code, ideas and feedback.
I [announced the project](https://dmsimard.com/2016/05/21/ara-an-idea-to-store-browse-and-troubleshoot-ansible-playbook-runs/) to the greater public with a [beta preview](https://www.youtube.com/watch?v=k3qtgSFzAHI).

ARA is one month old and has more than 200 commits today. Since the prototype, ARA has gained quite a few features. Amongst other things:

- Static generation of the web application, including feature parity with the dynamic web application
- Host fact recording and browsing
- A CLI client interface
- Support for aborted playbook runs and overall improvements around failure and exception handling
- A lot of unit and integration tests to ensure stability and prevent regressions ([we can even run Ansible's own integration test suite!](https://github.com/dmsimard/ara/commit/c1883013e8352eef68420e9136964547eb6a2e0c))

The OpenStack and the Ansible communities are excited with ARA's potential and opportunities to help them.
This fuels my motivation even further and I was already passionate about the project to help me in my own job to begin with.

If ARA can help other Ansible users make sense of their playbook runs, saying that I'm very happy about it would be an understatement.

### The next steps for ARA: To infinity and beyond
#### A proper development, contribution and issue tracking workflow.
The [OpenStack](http://www.openstack.org/) project has awesome [tooling, hosting, and testing infrastructure](http://docs.openstack.org/infra/system-config/).

Every day, developers send hundreds of commits for review across more than a hundred projects in [Gerrit](https://review.openstack.org/#/).
These will go through the [Zuul](https://github.com/openstack-infra/zuul) pipeline manager and eventually tested with [Jenkins](https://jenkins.io/) on ephemeral virtual machines. These virtual machines are managed by
[nodepool](http://docs.openstack.org/infra/system-config/nodepool.html) and hosted on OpenStack cloud resources [donated by different contributing companies](http://docs.openstack.org/infra/system-config/contribute-cloud.html).

OpenStack provides these resources under a program that was previously known as [StackForge](http://docs.openstack.org/infra/system-config/stackforge.html) for projects that are relevant to the OpenStack community and want to use the same development workflow.
It does not imply governance of the projects by the OpenStack foundation or the
OpenStack technical committee. The self-appointed core contributor
team continues to drive the direction of the project and decides what goes in (or
not).

OpenStack happens to host several projects that are not OpenStack
specific, a good example of which is [Jenkins Job Builder](https://github.com/openstack-infra/jenkins-job-builder) (JJB).
While JJB was created to scratch an itch in the OpenStack community, the
tool's development remained agnostic and today sees development and
usage from outside the OpenStack community.

There was significant interest to have ARA be part of the Ansible community umbrella as well.
It was a hard decision, but we ended up choosing the OpenStack community as it's home.

ARA, if all goes well, should be joining the OpenStack ecosystem [soon](https://review.openstack.org/#/c/321226/) and we'll update documentation with the contribution workflow once that is done.

#### A stable release
The break-neck pace of development in ARA until now has prevented us from declaring any sort of stable release beyond tagged development releases.

This doesn't mean ARA has a lot of bugs, it means the database schema is still changing quite a bit. Because of this, we have chosen to not develop automated database migrations and upgrades.
This helps us merge new features and fixes faster without database migration overhead while the project is not yet considered stable.

We have started identifying features which might require a database schema change.
Once we have ironed these out and the schema hasn't changed in a bit, it will be a good time to declare the schema stable and draft the first stable release of ARA.
When that first stable release is out, we will try to make sure that ARA can be upgraded without having to start your database from scratch.

#### An even better user experience
I'd love ARA's web application to have it's own distinct identity, look and feel. It's a fairly generic web application with very basic html, javascript and bootstrap CSS right now.

I'm really not a frontend guy. I sincerely hope that, over time, we can have skilled contributors help on that front.

#### More features and improvments based on what users need
I'm a demanding Ansible user and I'll be using ARA a lot, that's for sure.
The feature development has been mostly driven by what I've needed so far.

We have already received a lot of valuable feedback and comments about ARA.
I'd love this trend to continue so that we can improve ARA by adding or changing things we didn't think about.

If ARA doesn't do something you'd need, I want to know about it.
If ARA could do something better, I want to know about it.

We use [GitHub Issues](https://github.com/dmsimard/ara/issues) to help us keep track of things for the time being.

### See you at the stable release
The first stable release will be a great milestone for ARA. Let's do another recap once it's out !

Until then, come chat with us on #ara on freenode IRC and thanks for all the interest and support !
