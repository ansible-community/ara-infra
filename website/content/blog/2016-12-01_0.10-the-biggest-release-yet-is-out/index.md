---
author: "David Moreau Simard"
categories:
  - development
  - changelog
tags:
  - ansible
  - openstack
date: 2016-12-01
title: 0.10, the biggest release yet, is out !
slug: 0.10-the-biggest-release-yet-is-out
type: post
---

19 commits, 59 changed files, 2,404 additions and 588 deletions... and more than a month's on and off work.

[0.10 is out](https://github.com/openstack/ara/releases/tag/0.10.0) !

Where to get it ? Get started easily by [installing](http://ara.readthedocs.io/en/latest/installation.html) and [configuring](http://ara.readthedocs.io/en/latest/configuration.html) ARA.

I'm excited to tell you about this new release ! Here's a few highlights !

## An improved web application browsing experience

A lot of work has gone into the browsing experience: less clicks, more information, faster.

I am by no means a professional frontend developer and I would definitely love help on this front but I think things are definitely starting to look good !
I've added updated previews to the documentation:

- In [screenshots](https://ara.readthedocs.io/en/latest/faq.html#what-does-the-web-interface-look-like)
- In [video](https://www.youtube.com/watch?v=zT1l-rFne-Q&index=3&list=PLyLLwe4-L1ETFVoAogQqpn6s5prGKL5Ty) where we I showcase some playbook runs by the [OpenStack-Ansible](https://github.com/openstack/openstack-ansible) project.

It's awesome to see how much the interface has improved over time.

Curious ? Look at the [Alpha](https://www.youtube.com/watch?v=K3jTqgm2YuY&index=1&list=PLyLLwe4-L1ETFVoAogQqpn6s5prGKL5Ty) and [Beta](https://www.youtube.com/watch?v=k3qtgSFzAHI&list=PLyLLwe4-L1ETFVoAogQqpn6s5prGKL5Ty&index=2) video previews to see how far the project has come !

## Two new Ansible modules

This new release features two new Ansible modules, [ara_record](http://ara.readthedocs.io/en/latest/usage.html#using-the-ara-record-module) and [ara_read](http://ara.readthedocs.io/en/latest/usage.html#using-the-ara-read-module).

These allow you to respectively write and read persistent data that is made available throughout your playbook run and in the web interface.

For example:
```
---
- name: Test playbook
  hosts: localhost
  tasks:
    - name: Get git version of playbooks
      command: git rev-parse HEAD
      register: git_version

    - name: Record git version
      ara_record:
        key: "git_version"
        value: "{{ git_version.stdout }}"
```

That would make the git_version key/value available in the web interface and on the CLI.

And you can record different kind of data, too, see:

```
---
- ara_record:
    key: "{{ item.key }}"
    value: "{{ item.value }}"
    type: "{{ item.type }}"
  with_items:
    - { key: "log", value: "error", type: "text" }
    - { key: "website", value: "http://domain.tld", type: "url" }
    - { key: "data", value: '{ "key": "value" }', type: "json" }
```

Setting a type will make it so it's displayed accordingly in the interface.
More types will eventually be made available !

## Improved unit and integration testing coverage

In order to make sure ARA works well -- and keeps working well -- we need to have a good amount of unit and integration testing.

This coverage was improved since the previous release and additionnally, we now test against different versions of Ansible on both CentOS and Ubuntu.

## Database schema finally declared stable

Was ARA unstable before ? Not really.
However, at the rapid pace of development we were having with the project, we decided to avoid managing SQL migrations to avoid unnecessary overhead as the database schema was moving a lot.

I feel the database is fairly fleshed out at this point at any new modifications will be handled automatically when running ARA.

Any version of ARA >= 0.9.0 is considered a stable and managed database schema.

## This is great, let's make it better !

The feedback around ARA has been awesome, keep it coming !
A lot of the improvements that are part of this release is directly from ideas and comments provided by users.

If you want to come chat, learn or discuss about ARA, you'll find us on #ara on IRC in the freenode server !
