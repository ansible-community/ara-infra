---
author: "David Moreau Simard"
categories:
  - news
tags:
  - ansible
date: 2020-04-14
title: "ARA 0.16.7 and end of life for 0.x"
slug: ara-0.16.7-and-end-of-life-for-0.x
type: post
---

0.16.7 is a maintenance release for ARA 0.x.

Changes since 0.16.6:

- Fix typo in ara.setup.env for ANSIBLE_ACTION_PLUGINS ([pull request](https://github.com/ansible-community/ara/pull/97))
- Pin pyfakefs to <4 in order to avoid breaking python2 usage ([issue](https://github.com/ansible-community/ara/issues/118))
- Pin junit-xml to <=1.8 in order to avoid deprecation warnings in unit tests

## ARA 0.x end of life

The code base for ARA 0.x has not been actively maintained and developed
since 2018 and will officially reach end of life June 4th, 2019, one year
after the release of ARA 1.0.

Unless critical bugs are found between this release and June 4th, 0.16.7
will be the last supported release of the [0.x branch](https://github.com/ansible-community/ara/tree/stable/0.x).

Please use the latest version of ARA to benefit from the
new features and fixes.

## Want to try ARA ?

Have a look at the [quickstart](https://github.com/ansible-community/ara#quickstart) or
read the [installation](https://ara.readthedocs.io/en/latest/installation.html)
and [configuration](https://ara.readthedocs.io/en/latest/ansible-configuration.html)
documentation for more information.

## Want to contribute, chat or need help ?

ARA could use your help and we can also help you get started.
Please reach out !

The project community hangs out on [IRC and Slack](https://ara.recordsansible.org/community/).
