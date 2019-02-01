ansible-role-ara-server
=======================

This Ansible role provides a framework for installing one or many instances of
`ara-server <https://github.com/openstack/ara-server>`_ in a variety of
opinionated deployment topologies.

It is currently tested and supported against Ubuntu 18.04 and Fedora 29.

Role Variables
==============

See ``defaults/main.yaml``.

TL;DR
=====

This is what the role does by default out of the box:

- Installs required packages
- Creates a user for running ara-server
- Creates standard directories (``/var/lib/ara``, ``/etc/ara``, ``/var/log/ara``, ``/var/www/ara-server``)
- Retrieves ara-server from source
- Installs ara-server in a virtualenv
- Generates a random secret key if none are provided or already configured
- Configures ``/etc/ara/settings.yaml``
- Runs SQL migrations (``ara-manage migrate``)
- Installs gunicorn in the same virtualenv as ara-server
- Sets up a systemd unit file for running ara-server with gunicorn
- Collects static files (``ara-manage collectstatic``) into ``/var/www/ara-server``
- Includes the ``ara_frontend_nginx`` role to install and configure nginx as a reverse proxy to gunicorn

About deployment topologies
===========================

This Ansible role is designed to support different opinionated topologies that
can be selected with role variables.

For example, the following role variables are used to provide the topology from
the ``TL;DR`` above:

- ``ara_server_install_method: source``
- ``ara_server_wsgi_server: gunicorn``
- ``ara_server_database_engine: django.db.backends.sqlite3``
- ``ara_server_frontend_server: nginx``

The intent is that as the role gains support for other install methods,
wsgi servers, frontend servers or database engines, it will be possible to
mix and match according to preference or requirements.

Perhaps ara-server could be installed from pypi and run with uwsgi, nginx and mysql.
Or maybe it could be installed from distribution packages and set up to run with apache, mod_wsgi and postgresql.
Or any combination of any of those.

Copyright
=========

::

    Copyright (c) 2019 Red Hat, Inc.

    ARA Records Ansible is free software: you can redistribute it and/or modify
    it under the terms of the GNU General Public License as published by
    the Free Software Foundation, either version 3 of the License, or
    (at your option) any later version.

    ARA Records Ansible is distributed in the hope that it will be useful,
    but WITHOUT ANY WARRANTY; without even the implied warranty of
    MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
    GNU General Public License for more details.

    You should have received a copy of the GNU General Public License
    along with ARA Records Ansible. If not, see <http://www.gnu.org/licenses/>.
