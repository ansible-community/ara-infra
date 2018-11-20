---
kind: home
---

# What's ARA ?

ARA Records Ansible playbook runs and makes the recorded data available and intuitive for users and systems.

It makes your Ansible playbooks easier to understand and troubleshoot.

![reports](static/reports.png)
<br>
<br>

### ARA is free and open source

ARA is an [OpenStack](https://www.openstack.org) community project and all of it's [components](https://github.com/openstack?q=ara)
are free and open source under the GPLv3 license.

You can participate in [code reviews](https://review.openstack.org/#/q/project:%255Eopenstack/ara.*+OR+project:openstack/ansible-role-ara)
and learn how you can contribute your first patch in the [contributors documentation](https://ara.readthedocs.io/en/latest/contributing.html).
<br>
<br>

### ARA is simple and easy to use

Simplicity is a core feature in ARA.
It does one thing and it does it well: reporting on your Ansible playbooks.

[Install ARA](http://ara.readthedocs.io/en/stable/installation.html), tell [Ansible](https://ara.readthedocs.io/en/latest/configuration.html)
to use it and your next playbook will be recorded. That's it !

Read more about the project's core values in the [manifesto](https://ara.readthedocs.io/en/stable/manifesto.html).
<br>
<br>

### ARA is tested, stable and production ready

Each new commit to ARA is gated against a series of [unit and integration tests](https://github.com/openstack/ara#contributing-testing-issues-and-bugs)
against different Linux distributions and versions of Ansible in order to prevent regressions.

ARA is used to record more than a [million playbooks a month](http://superuser.openstack.org/articles/scaling-ara-ansible/) from the OpenStack community alone.

It works.
<br>
<br>

### ARA is offline and decentralized by default

Running Ansible from your laptop ? No problem.

You can browse your ARA reports locally from a sqlite database without ever leaving the comfort of localhost.

Need to scale with real [web and application servers](https://ara.readthedocs.io/en/stable/webserver.html) or use a
database server like [MySQL or PostgreSQL](https://ara.readthedocs.io/en/stable/configuration.html#ara-database) ?

You can do that too.
<br>
<br>

# Getting started

```
# Install ARA from PyPi
$ pip install ara

# Set up Ansible to use ARA
$ export ANSIBLE_CALLBACK_PLUGINS=$(python -m ara.setup.callback_plugins)

# Run your Ansible playbook as usual
$ ansible-playbook myplaybook.yml

# Start the ARA standalone webserver
$ ara-manage runserver

# Browse http://127.0.0.1:9191
```
