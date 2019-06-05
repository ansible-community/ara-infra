---
author: "David Moreau Simard"
categories:
  - development
tags:
  - ansible
  - openstack
date: 2019-01-16
title: "Introducing new projects for the upcoming 1.0 release"
slug: introducing-new-projects-for-the-upcoming-1.0-release
type: post
---

Attempting to release a version 1.0 for ARA Records Ansible has been quite an
adventure that started more than a year ago.

The general vision hasn't changed since then and the status updates from 2017
are still relevant from that perspective.

Hundreds of commits later, the code underneath what will be released as ARA 1.0
has been almost entirely re-written from scratch, though.

We've been making steady progress with the help of two new core contributors
(thanks Florian and Guillaume!) and we're excited to share some of the things
we have been working on.

We'll be providing updates on a more regular basis as we draw closer to the
official release of 1.0 but I first wanted to introduce how ARA 1.0 has been
split into different projects.

# Different projects

In 1.0, ARA will no longer be a single project requiring you to install
dependencies that you might not be interested in for a wide array of use cases.

For example, in 0.x, you would need to install the web application dependencies
even though you might just be after the callback plugin.

As things stand today, ARA 1.0 is composed of four individual projects which
lets you install them together or independently of one another depending on
your use case.

You can find more information on the general workflow and interaction between
the components in the [documentation](https://ara-server.readthedocs.io/en/latest/arch.html).

## ara-server

ara-server ([github](https://github.com/ansible-community/ara-server)) is a new python 3
application built with [django](https://www.djangoproject.com/) and
[django-rest-framework](https://www.django-rest-framework.org/).

It provides the database models as well as a brand new REST API for
integrating ARA's playbook execution results into your workflows.

If you want to see what the API looks like in practice, there is a live demo
available [here](https://api.demo.recordsansible.org/api/v1).

This web interface, provided out of the box by django-rest-framework, allows you
to navigate or query the API directly from your web browser.

We're not quite ready to call the API stable yet but we're getting there.
As we start deploying and using the API ourselves, we expect to improve and
fine tune things before the official release.

An important consideration we had when developing this new API is that it should
not be necessary for users to stand up an API server.

This brings us to the next component we'll talk about, ara-clients.

## ara-clients

ara-clients ([github](https://github.com/ansible-community/ara-clients)) is the project
that provides the python API clients for communicating with the API.

It currently ships two clients, one that works without needing to stand up an
API server, ``AraOfflineClient`` and one that expects a remote HTTP API
endpoint, ``AraHttpClient``.

The clients have the same interface and they take care of abstracting the
differences when running offline or online.

In a nutshell, the current implementation of the offline client spawns an
ephemeral API server instance so you don't need to.

Using these clients with the API is designed to be as straightforward as possible.
Here's a simple example from the
[documentation](https://ara-server.readthedocs.io/en/latest/using_api_clients.html):

{{< highlight python >}}
#!/usr/bin/env python3
# Import the client
from ara.clients.http import AraHttpClient

# Instanciate the HTTP client with an endpoint where ara-server is listening
client = AraHttpClient(endpoint="https://api.demo.recordsansible.org")

# Get a list of failed playbooks
# /api/v1/playbooks?status=failed
playbooks = client.get("/api/v1/playbooks", status="failed")

# If there are any failed playbooks, retrieve their failed results
# and provide some insight.
for playbook in playbooks["results"]:
    # Retrieve results for this playbook
    # /api/v1/results?playbook=<:id>&status=failed
    results = client.get("/api/v1/results", playbook=playbook["id"], status="failed")

    # Iterate over failed results to get meaningful data back
    for result in results["results"]:
        # Get the task that generated this result
        # /api/v1/tasks/<:id>
        task = client.get(f"/api/v1/tasks/{result['task']}")

        # Get the file from which this task ran from
        # /api/v1/files/<:id>
        file = client.get(f"/api/v1/files/{task['file']}")

        # Get the host on which this result happened
        # /api/v1/hosts/<:id>
        host = client.get(f"/api/v1/hosts/{result['host']}")

        # Print something useful
        print(f"Failure on {host['name']}: '{task['name']}' ({file['path']}:{task['lineno']})")
{{< /highlight >}}

Running this bit of code against the current demo API provides the following
output:

```
Failure on localhost: 'smoke-tests : Record with no key' (/home/zuul/src/git.openstack.org/openstack/ara-infra/tests/integration/roles/smoke-tests/tasks/ara-ops.yaml:166)
Failure on localhost: 'smoke-tests : Record with no value' (/home/zuul/src/git.openstack.org/openstack/ara-infra/tests/integration/roles/smoke-tests/tasks/ara-ops.yaml:177)
Failure on localhost: 'smoke-tests : Record with invalid type' (/home/zuul/src/git.openstack.org/openstack/ara-infra/tests/integration/roles/smoke-tests/tasks/ara-ops.yaml:188)
Failure on localhost: 'smoke-tests : Return false' (/home/zuul/src/git.openstack.org/openstack/ara-infra/tests/integration/roles/smoke-tests/tasks/test-ops.yaml:25)
Failure on localhost: 'fail' (/tmp/ara-integration-tests/ara-infra/tests/integration/failed.yaml:22)
```

This simple example illustrates how you can search for things with the API and
access every bit of data from your playbook executions.
Playbook and role files, host facts, task metrics, granular and detailed
results, everything is there and organized to make it easy to work with.

These clients are the same ones used by the ARA Ansible callback plugin as well
as the ``ara_record`` Ansible module in order to communicate with the API.
These are part of the next project from 1.0 we'll talk about: ara-plugins.

## ara-plugins

ara-plugins ([github](https://github.com/ansible-community/ara-plugins)) is the project
that contains Ansible modules and plugins for ARA.

Ansible provides a [callback plugin interface](https://docs.ansible.com/ansible/latest/plugins/callback.html)
in order to let users hook into certain events that occur during the playbook
execution such as ``v2_playbook_on_start``, ``v2_playbook_on_task_start``,
``v2_runner_on_ok``, ``v2_runner_on_failed``, etc.

ara-plugins provides a callback plugin which leverages this interface to save
everything to a database by sending it to the API throughout the playbook run.

The project also contains an Ansible module called ``ara_record`` which
can be used to record anything and associate it with the playbook report.

We've seen users record things like git commit hashes, links to artifacts or
logs and other data relevant to their use cases.

The callback and the ara_record module are not new to ARA 1.0 but they have been
updated to use the new API.
They are basically the only components that have not been re-written from scratch.

## ara-web

ara-web ([github](https://github.com/ansible-community/ara-web)) will be the new web
interface project in ARA 1.0.

It is a modern javascript interface built with [React](https://reactjs.org/)
and [patternfly-next](https://github.com/patternfly/patternfly-next), the
upcoming release of [Patternfly](https://www.patternfly.org/).

We are iterating on the actual web interface right now and a lot of technical
problems are solved. For example:

- You can configure which API server the built-in client will connect to
- It has it's own API client implementation for querying ara-server and parsing it's replies
- It has jobs for running unit and build integration tests on every patch
- We even get drafts of the dashboard during code reviews (more on that in another blog post)

Want to know what it looks like ? There's a [live demo](https://web.demo.recordsansible.org).
The live demo is stateless -- it relies entirely on the API live demo to retrieve
data for displaying the reports.

## But wait, there's more !

There are two other projects that aren't about strictly recording Ansible, but
they're about helping us recording Ansible with Another Recursive Acronym.

### ara-infra

ara-infra ([github](https://github.com/ansible-community/ara-infra)) is a satellite
project for ARA's infrastructure related things.

So far, it contains:

- The source for this website and blog
- The Ansible playbooks and roles to deploy it from scratch
- The integration tests for the website deployment as well as the CI jobs to run them

This is also where we will eventually find integration tests common to all the
ARA projects as well as the jobs to update the API and web demos after each commit.

### ansible-role-ara

ansible-role-ara ([github](https://github.com/openstack/ansible-role-ara)) wants
to be able to install and configure ARA in a few different standardized deployment
scenarios:

- Inside a virtual environment (or not)
- From source or from distribution packages (when available)
- apache with mod_wsgi
- With nginx+gunicorn
- With nginx+uwsgi

Supporting these deployment scenarios would be a very good asset for integration
testing ARA itself.

# Coming soon (™)

This is all coming soon. It's not a matter of days but it's not years either !

If you would like to contribute code, feedback, documentation or even help test
the upcoming release, please reach out ! It could allow us to ship sooner :)

We're on [Twitter](https://twitter.com/ARecordsAnsible),
[#ara](http://webchat.freenode.net/?channels=%23ara) on the freenode IRC
network and on [Slack](https://join.slack.com/t/arecordsansible/shared_invite/enQtMjMxNzI4ODAxMDQxLWU4MmZhZTI4ZjRjOTUwZTM2MzM3MzcwNDU1YzFmNzRlMzI0NTUzNDY1MWJlNThhM2I4ZTViZjUwZTRkNTBiM2I).

See you around !
