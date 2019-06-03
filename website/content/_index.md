---
kind: home
---

# What's ARA ?

ARA Records Ansible playbooks and makes them easier to understand and troubleshoot.

![reports](static/reports.png)

ARA saves playbook results to a local or remote database by using an Ansible
callback plugin and provides an API to integrate this data in tools and interfaces.

- For the API as well as the Ansible components, see [ara](https://github.com/ansible-community/ara)
- For the web client interface, see [ara-web](https://github.com/ansible-community/ara-web)

## ARA is simple and easy to use

Simplicity is a core feature in ARA.
It does one thing and it does it well: reporting on your Ansible playbooks.

Here's how you can get started from scratch with sane defaults:

```bash
# Create a python3 virtual environment and activate it so we don't conflict
# with system or distribution packages
python3 -m venv ~/.ara/virtualenv
source ~/.ara/virtualenv/bin/activate

# Install Ansible, ARA and it's API server dependencies
pip install ansible ara[server]

# Tell Ansible to use the ARA callback plugin
export ANSIBLE_CALLBACK_PLUGINS="$(python -m ara.setup.callback_plugins)"

# Run your playbook as usual
ansible-playbook playbook.yml
```

If nothing went wrong, your playbook data should have been saved in a local
database at ``~/.ara/server/ansible.sqlite``.

You can browse this data through the API by executing ``ara-manage runserver``
and pointing your browser at http://127.0.0.1:8000/.

That's it !

## Live demos

You can find live demos deployed by the built-in [ara_api](https://ara.readthedocs.io/en/feature-1.0/ansible-role-ara-api.html)
and [ara_web](https://ara.readthedocs.io/en/feature-1.0/ansible-role-ara-web.html)
Ansible roles at https://api.demo.recordsansible.org and https://web.demo.recordsansible.org.

## ARA is free and open source

ARA is free and open source under the GPLv3 license.

The code review and CI infrastructure is hosted by [OpenDev](https://opendev.org).

You can participate in [code reviews](https://review.opendev.org/#/q/project:%255Erecordsansible/.*)
and learn how you can contribute your first patch in the [contributors documentation](https://ara.readthedocs.io/en/feature-1.0/contributing.html).

## ARA is tested, stable and production ready

Each new commit to ARA is gated against a series of unit and integration tests
against different Linux distributions and versions of Ansible in order to
prevent regressions.

ARA is used to record more than a [million playbooks a month](http://superuser.openstack.org/articles/scaling-ara-ansible/) from the OpenStack community alone.

It works.

## ARA is offline and decentralized by default

Running Ansible from your laptop ? No problem.

You can browse your ARA reports locally from a sqlite database without ever leaving the comfort of localhost.

Need to aggregate data from multiple locations ?
You can run an API server and hook it up to a database engine like
[PostgreSQL or MySQL](https://ara.readthedocs.io/en/feature-1.0/api-configuration.html#ara-database-engine).
