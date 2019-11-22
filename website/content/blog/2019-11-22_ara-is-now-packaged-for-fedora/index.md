---
author: "David Moreau Simard"
categories:
  - news
tags:
  - ansible
date: 2019-11-22
title: "ARA is now packaged for Fedora !"
slug: ara-is-now-packaged-for-fedora
type: post
---

We are happy to announce that ARA has successfully been included in
[Fedora](https://getfedora.org/) !

The latest version ([1.2.0](https://ara.recordsansible.org/blog/2019/11/06/announcing-the-release-of-ara-records-ansible-1.2/))
is available for Fedora 30, Fedora 31 as well as Rawhide (Fedora 32).

You can find the source for the packaging files on [src.fedoraproject.org](https://src.fedoraproject.org/rpms/ara/tree/master)
and the builds on [koji.fedoraproject.org](https://koji.fedoraproject.org/koji/packageinfo?packageID=24394).

Contributions to improve the packaging are welcome !

## How to get started

If you are using Ansible from Fedora packages, you can start recording playbooks
with ARA easily:

```
# Install ARA's Ansible plugins and the API server dependencies
dnf install ara ara-server

# Configure Ansible to load the ARA callback plugin
export ANSIBLE_CALLBACK_PLUGINS=$(python3 -m ara.setup.callback_plugins)

# Run your playbooks as usual
ansible-playbook playbook.yml

# Start the embedded server
ara-manage runserver

# That's it !
```

You can see what it looks like in this small recording:

<script id="asciicast-HrXjFmvCygxaPbmVA0u6ZXAKL" src="https://asciinema.org/a/HrXjFmvCygxaPbmVA0u6ZXAKL.js" async></script>

If you'd like to see the built-in web interface looks like, you can find a live
demo on [api.demo.recordsansible.org](https://api.demo.recordsansible.org).

## Want to contribute, chat or need help ?

If you would like to help with packaging ARA for other distributions, please
reach out !

The project community hangs out on [IRC and Slack](https://ara.recordsansible.org/community/).
