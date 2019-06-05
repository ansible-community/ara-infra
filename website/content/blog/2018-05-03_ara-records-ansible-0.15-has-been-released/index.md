---
author: "David Moreau Simard"
categories:
  - development
  - changelog
tags:
  - ansible
  - openstack
date: 2018-05-03
title: "ARA Records Ansible 0.15 has been released"
slug: ara-records-ansible-0.15-has-been-released
type: post
---

I was recently writing that ARA was open to limited development for the stable
release in order to improve the performance for larger scale users.

This limited development is the result of this 0.15.0 release.

{{< tweet 986675784478752770 >}}

# Changelog for ARA Records Ansible 0.15.0

The following are highlights from the 0.15.0 release.
The full list of changes between 0.14.6 and 0.15.0 are available on [GitHub](https://github.com/ansible-community/ara/compare/0.14.6...0.15.0).

- ARA: Ansible Run Analysis has been [rebranded](https://dmsimard.com/2018/02/23/rebranding-ansible-run-analysis-to-ara-records-ansible/)
  to ARA Records Ansible (Another Recursive Acronym)
- Significant improvements to memory usage and performance when running ARA as
  a WSGI application with ``ara-wsgi`` or ``ara-wsgi-sqlite``.
- Resolved an issue where the ``ara-wsgi-sqlite`` middleware could serve a
  cached report instead of the requested one
- Added support for configuring the ``SQLALCHEMY_POOL_SIZE``,
  ``SQLALCHEMY_POOL_TIMEOUT`` and ``SQLALCHEMY_POOL_RECYCLE`` parameters.
  See [configuration documentation](https://ara.readthedocs.io/en/latest/configuration.html#parameters-and-their-defaults) for more information.
- Logging was fixed and improved to provide better insight when in DEBUG level.
- Vastly improved the default [logging configuration](https://github.com/ansible-community/ara/blob/3c91da40871e50fa2854231d54f298ed996d1da6/ara/config/logger.py#L27-L78).
  ARA will create a default logging configuration file in ``~/.ara/logging.yml`` that you can customize, if need be.
  Deleting this file will make ARA create a new one with updated defaults.
- Added python modules to help configure Ansible to use ARA, for example,
  ``python -m ara.setup.callback_plugins`` will print the path to ARA's callback plugins.
  You can find more examples in the [configuration documentation](https://ara.readthedocs.io/en/latest/configuration.html).
- Implemented a workaround for fixing a race condition where an ``ansible-playbook`` command
  may be interrupted after the playbook was recorded in the database but before playbook file was saved.
- Flask 0.12.3 was [blacklisted](https://github.com/ansible-community/ara/commit/87272840bfc8b4c5db10593e47884e33a0f4af40)
  from ARA's requirements, this was a broken release.
- The ARA CLI can now be called with "python -m ara". This can be useful if you
  need to specify a specific python interpreter, for example.
- Updated and improved integration tests across different operating systems,
  python2 and python3 with different versions of Ansible. The full test matrix
  is available in the [README](https://github.com/ansible-community/ara#contributing-testing-issues-and-bugs).

# Known issue: memory usage

Please note that there is a known issue regarding high memory usage when
browsing reports in the web application for **very large** reports.

For the time being, we have put mitigations in place in order to prevent very
high memory usage but this might be insufficient for large scale users.

{{< tweet 984265830052651008 >}}

If you'd like to help and contribute to resolve this, please reach out.

# A disclaimer

As with previous releases of the 0.x series, this new version comes with a 
warning that we are not currently planning to provide backwards/forwards
compatibility with the next major release of ARA, 1.0.

In practice, this means that we are trying to focus all of our current
contribution efforts towards making 1.0 as great as possible without burdening
our limited development resources with keeping backwards and forwards
compatibility.
This means your existing databases will not be upgradable to the new version so
you will need to start from a new database.

One thing you don't have to worry about is that we are serious about keeping
the project as simple as possible, in line with the project's [manifesto and core values](https://ara.readthedocs.io/en/latest/manifesto.html).
ARA 1.0 will work the same way as it did before: ``pip install ara``, ``export ANSIBLE_CALLBACK_PLUGINS=$ara_location/plugins/callbacks`` and you're good to go.

1.0 will provide additional features such as a revamped database, a backend
rewritten from scratch, an improved frontend and a full REST API.

You can look at some of the work we have been doing for this new version [here](https://dmsimard.com/categories/ara/).

There are a lot exciting opportunities where you can contribute to the development
of 1.0, please reach out to me (dmsimard) if you're interested in helping !

You can come chat with us on [IRC or on Slack](https://github.com/ansible-community/ara#contributing-testing-issues-and-bugs).
