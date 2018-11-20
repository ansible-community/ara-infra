---
author: "David Moreau Simard"
categories:
  - development
tags:
  - ansible
  - openstack
date: 2016-05-21
title: "ARA: An idea to store, browse and troubleshoot Ansible Playbook runs"
slug: an-idea-to-store-browse-and-troubleshoot-ansible-playbook-runs
type: post
---

# The context
[Ansible](https://www.ansible.com/) can be used for a lot of things and it's
grown pretty popular for managing servers and their configuration.

In the [RDO](https://www.rdoproject.org/) and
[OpenStack](https://www.openstack.org/) communities, Ansible is heavily used to
deploy or test OpenStack through Continuous Integration (CI). Projects like
[TripleO-Quickstart](https://github.com/openstack/tripleo-quickstart),
[WeIRDO](https://github.com/redhat-openstack/weirdo),
[OpenStack-Ansible](https://github.com/openstack/openstack-ansible) or
[Zuul v3](https://specs.openstack.org/openstack-infra/infra-specs/specs/zuulv3.html#ansible)
are completely driven by Ansible.

In the world of automated continuous integration, it's not uncommon to have
hundreds, if not thousands of jobs running every day for testing, building,
compiling, deploying and so on.

Keeping up with a large amount of Ansible runs and their outcome, not
just in the context of CI, is challenging.

# The idea
[ARA](https://github.com/dmsimard/ara) is an idea I came up with to try
and make Ansible runs easier to visualize, understand and troubleshoot.

ARA is three things:

1. An [Ansible callback plugin](https://ara.readthedocs.io/en/latest/configuration.html#ansible) to record playbook runs into a local or remote database
2. A [CLI client](https://ara.readthedocs.io/en/latest/usage.html#querying-the-database-with-the-cli) to query the database
3. A [web interface](https://ara.readthedocs.io/en/latest/faq.html#what-does-the-web-interface-look-like) to visualize the database

ARA organizes recorded playbook data in a way to make it intuitive for you to
search and find what you're interested for as fast and as easily as possible.

It provides summaries of task results per host or per playbook.

It allows you to filter task results by playbook, play, host, task or by the
status of the task.

With ARA, you're able to easily drill down from the summary view for the results
you're interested in, whether it's a particular host or a specific task.

Beyond browsing a single ansible-playbook run, ARA supports recording and
viewing multiple runs in the same database.

This allows you to, for example, recognize patterns (ex: this particular host
is always failing this particular task) since you have access to data from
multiple runs.

ARA is an open source project available on [Github](https://github.com/dmsimard/ara) under the Apache v2 license.
[Documentation](https://ara.readthedocs.io/en/latest/) and
[frequently asked questions](https://ara.readthedocs.io/en/latest/faq.html) are available on readthedocs.io.

# Why ?
As I mentioned before, the vast majority of the RDO CI is powered by Ansible.
When a job build fails, I have to look at one of these Jenkins
[console logs](https://dmsimard.com/files/ansible-jenkins.txt) that's >8000
lines long. If it doesn't crash my browser, I get to dig across the large
amount of output to try and figure out what went wrong in the job build.

When you're testing OpenStack trunk, you're going to be troubleshooting a *lot*
of those large failed jobs and it's painful.
Over time, I've (*unfortunately*) gotten used to it and got pretty good, actually.
However, it still takes me a non negligible amount of time just to find where
Ansible failed to know where to start searching for in the logs.

It's also definitely a nightmare when someone else wants to look at the jobs 
to try and understand what happened.

ARA solves that painpoint - and many others - by making it easier to
browse the results of a playbook.
 
### Other attempts
To try and help us before ARA was born, we leveraged two callbacks to
try and help us parse the Ansible Playbook output.

The first is [human_log.py](https://gist.github.com/cliffano/9868180) which
helps pretty-printing output from tasks like "command" or "yum".
We also have [profile_tasks](https://github.com/jlafon/ansible-profile/blob/master/callback_plugins/profile_tasks.py)
that is built-in and helps by showing how much time each task took.

These callbacks are definitely helpful for small playbooks or playbooks that
contain small or short-running tasks.
On long-running playbooks with a large amount of output, they almost make matters
worse by adding even more output into the task results.

# How do I get started with ARA ? 
I've tried to do simple, yet effective documentation on how to get started
with ARA.

### 1) Install ARA
Documentation: [https://ara.readthedocs.io/en/latest/installation.html](https://ara.readthedocs.io/en/latest/installation.html)

First, you'll need to install some packaged dependencies and then you
can install ARA from source or from pip.

For example on a CentOS server:

    yum -y install gcc python-devel libffi-devel openssl-devel
    pip install ara

### 2) Configure the callback
Documentation: [https://ara.readthedocs.io/en/latest/configuration.html#ansible](https://ara.readthedocs.io/en/latest/configuration.html#ansible)
([What's an Ansible Callback ?](https://ara.readthedocs.io/en/latest/faq.html#what-s-an-ansible-callback))

The configuration of the callback is simple and seemless. You want to
add the following to your ansible.cfg file:

    [defaults]
    callback_plugins = /usr/lib/python2.7/site-packages/ara/callback
    
    # Or, if using a virtual environment, for example
    
    [defaults]
    callback_plugins = $VIRTUAL_ENV/lib/python2.7/site-packages/ara/callback

### 3) Run a playbook with ansible-playbook
Run your favorite playbook !

### 4.1) Browse your data through the CLI
Documentation: [https://ara.readthedocs.io/en/latest/usage.html#querying-the-database-with-the-cli](https://ara.readthedocs.io/en/latest/usage.html#querying-the-database-with-the-cli)

    $ ara result list
    +--------------------------------------+-------------+----------------+--------+---------------+----------------------------+----------------------------+
    | ID                                   | Host        | Task           | Status | Ignore Errors | Time Start                 | Time End                   |
    +--------------------------------------+-------------+----------------+--------+---------------+----------------------------+----------------------------+
    | a73efa33-0d1e-4a7d-8e28-a76fa93b9377 | localhost   | Debug thing    | ok     | False         | 2016-05-21 14:42:24.794070 | 2016-05-21 14:42:24.837268 |
    +--------------------------------------+-------------+----------------+--------+---------------+----------------------------+----------------------------+
                                                                                                                                                                                                                                                                          
    $ ara result show a73efa33-0d1e-4a7d-8e28-a76fa93b9377
    +---------------+----------------------------------------------------+
    | Field         | Value                                              |
    +---------------+----------------------------------------------------+
    | ID            | a73efa33-0d1e-4a7d-8e28-a76fa93b9377               |
    | Host          | localhost                                          |
    | Task          | Debug thing (d04a5828-d32f-4349-89f1-39d7400b328f) |
    | Status        | ok                                                 |
    | Ignore Errors | False                                              |
    | Time Start    | 2016-05-21 14:42:24.794070                         |
    | Time End      | 2016-05-21 14:42:24.837268                         |
    +---------------+----------------------------------------------------+

### 4.2) Browse your data through the web interface
Documentation: [https://ara.readthedocs.io/en/latest/usage.html#browsing-the-web-interface](https://ara.readthedocs.io/en/latest/usage.html#browsing-the-web-interface)
([What does the web UI look like ?](https://ara.readthedocs.io/en/latest/faq.html#what-does-the-web-interface-look-like))

Fire off the bundled webserver:

    $ ara-manage runserver
     * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)

And use your favorite browser.

# There's no step five !
We're all done here. That's the gist of it.

A lot of effort was made towards making ARA as simple to install,
configure and use as possible. It is meant to be able to run from start
to finish locally but it is also powerful enough if you'd like to
aggregate runs into a central server.

### Discussing or contributing to ARA
If you'd like to use ARA or contribute to the project, definitely feel
free ! Feedback, comments, ideas and suggestions are quite welcomed as
well.

I hang out in the **#ara** channel on freenode IRC if you want to come chat about ARA !

Special thanks to [Lars Kellogg-Stedman](http://blog.oddbit.com/) for the early feedback on the project, ideas and code contributions.
He was very helpful in fleshing and maturing the idea into something better.
