---
author: "David Moreau Simard"
categories:
  - news
tags:
  - ansible
date: 2019-11-18
title: "ARA 0.16 maintenance release and eventual end of life"
slug: ara-0.16-maintenance-release-and-eventual-end-of-life
type: post
---

ARA 0.16.6 is a maintenance release for users that are not yet using ARA 1.x.

Changes since 0.16.5:

- Fixed web application crash due to encoding/decoding of binary non-ascii
  content in task results
- The [sqlite middleware](https://ara.readthedocs.io/en/stable-0.x/advanced.html)
  was adapted to support running under gunicorn as well as mod_wsgi
- ``python -m ara.setup.env`` now returns commands that use bash expansion to
  take into account existing environment variables

## Eventual end of life for ARA 0.x

All new feature and development effort for more than a year has been spent on
the master branch of ARA which is the basis of version 1.x releases.

In fact, 0.x and 1.x are vastly different code bases at this point.
They are diverging by more than 400 commits and 1.x features a completely
re-written backend which includes many new features, notably a new REST API.

Users are encouraged to try the latest release of ARA, currently [1.2.0](https://ara.recordsansible.org/blog/2019/11/06/announcing-the-release-of-ara-records-ansible-1.2/),
and create issues on [GitHub](https://github.com/ansible-community/ara/issues)
if they encounter any problems or missing features.

ARA 0.16.6 could be the last release of ARA 0.x if no major issues are found.

## Want to try the latest release ?

Have a look at the [quickstart](https://github.com/ansible-community/ara#quickstart) or
read the [installation](https://ara.readthedocs.io/en/latest/installation.html)
and [configuration](https://ara.readthedocs.io/en/latest/ansible-configuration.html)
documentation for more information.

## Want to contribute, chat or need help ?

ARA could use your help and we can also help you get started.
Please reach out !

The project community hangs out on [IRC and Slack](https://ara.recordsansible.org/community/).
