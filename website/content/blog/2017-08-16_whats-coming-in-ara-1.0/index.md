---
author: "David Moreau Simard"
categories:
  - development
tags:
  - ansible
  - openstack
date: 2017-08-16
title: "What's coming in ARA 1.0 ?"
slug: whats-coming-in-ara-1.0
type: post
---

Not long ago, I wrote that ARA: Ansible Run Analysis had it's first [birthday](https://dmsimard.com/2017/05/08/ara-is-one-year-old-a-look-back-at-the-past-year/).

It was an important milestone and it was a great opportunity to reflect back on where
the project was coming from and think about what we needed to do in the future.

Just for fun, let's look at what I had written back in May to summarize what
was probably coming:

- ~~Python 3 compatibility~~
  - This is done and was shipped in [ARA 0.14](https://github.com/openstack/ara/releases/tag/0.14.0)!
    It was a lot of work but ARA is now working and is fully tested against both python2.7 and python3.5.
- ~~An implementation in Zuul~~
  - This is in [progress](https://review.openstack.org/#/c/487853/) ! It's no longer a question of "if" but rather "how".
    A lot of improvements in ARA 1.0 is geared towards making this integration easier (amongst other things) and we'll
    have a lot of discussions about it at the upcoming [OpenStack project team gathering](https://www.openstack.org/ptg/).
- ARA and [Ansible Tower](https://www.ansible.com/tower) working together ?
  - I'm excited to see the opportunities with Tower but it's not open source yet !
    We know it's coming soon and [latest estimates](https://www.reddit.com/r/ansible/comments/6jat04/my_review_of_ansiblefest_london_2017/djdrjgk/)
    put us at a release within the next year.

## ARA 1.0: An important version

ARA 1.0 will be the largest and most significant release ever since the project
was created.

For me, 1.0 is not just any number. I attach a lot of weight and importance to
it.

First, it signals a very clear message that I feel ARA is battle tested and stable
enough for production use.
Users and deployments, both small and large, have proven me that ARA can be relied on.

Second, it means that we are making backwards incompatible changes, especially
around the database schema: more on that later.

And finally, we are completely re-writing the backend and it's logic, both of which
had been implemented in the very early days when prototyping ARA.
This new foundation will help ARA move forward faster and make integration
easier for users and systems.

Development of ARA 1.0 is not finished yet.
You can follow the progress and the code reviews [here](https://review.openstack.org/#/q/project:openstack/ara+branch:feature/1.0)
or by hanging out in IRC in #ara on the freenode server.

Here's some highlights about things that have already landed or will land in this next release.

## What's coming: decoupling components and a revamped backend

ARA will have lived until 1.0 on a backend implementation that was born out of the
prototype and proof of concept more than a year ago. It has served us well but it was
no longer appropriate for the scope of the project.

One of the biggest flaws of the previous backend implementation was that the
different components were coupled and also imported and queried the database model directly.

For example, when using ARA in a centralized fashion, this meant:

- Installing the callback, module, CLI and Ansible on the web server(s) hosting the web application
- Installing the web application (and all it's dependencies) when only requiring the CLI, callback or modules
- Putting/sharing a username/password for a database connection string to send data back to a central database

From a developer perspective, it also meant that we had no abstraction to the database and needed
to know and understand how the different components worked to introduce new features.
It also meant understanding how to query the database directly and code duplication
because every feature had their own database queries.

The backend is being essentially re-written from scratch in order to leverage a
new python and REST API.
This layer of abstraction will make things easier to maintain and develop against.

## What's coming: a python and REST API

What is probably the biggest endeavour of the 1.0 release is the introduction of python and REST APIs.

We are doing this for the project's own consumption in the first place to decouple the
components and re-write the backend.
However, I'm curious to see what users and developers might come up with !

Something that was important for me when designing the API was to keep ARA simple.
The introduction of an API should not mean that users suddenly have to run a web server in order to collect data.
It shouldn't also be a requirement when running the web application because the reports can be
generated to static HTML.

Before 1.0, all you had to do to get started was to [install ARA](http://ara.readthedocs.io/en/latest/installation.html) and configure the [Ansible callback path](http://ara.readthedocs.io/en/latest/configuration.html).
This will not change and the default use case remains local and offline data collection with no additional
set up or configuration steps.

The approach I am taking with the two APIs is to make them identical with the exact
same interface. In fact, the python API is simply an internal passthrough to the REST API
with no HTTP calls involved. This will make it easier to use, develop, test and maintain because of the
feature parity that is available by default.

Here's some examples to show the APIs are the same -- these examples do and return the exact same thing.

**For retrieving data**:

```
# Get the playbook with 'id' 1 over http
curl -H "Content-Type: application/json" -X GET <ara>/api/v1/playbooks/1

# Get the playbook with 'id' 1 with the python API
from ara.api.playbooks import PlaybookApi
playbook = PlaybookApi().get(id=1)
```

**For creating data**:

```
# Create a play in a playbook over http
curl -H "Content-Type: application/json" -X POST -d '{ "playbook_id": 1, "name": "Play name", "started": "1970-08-14T00:52:49.570031" }' <ara>/api/v1/plays/

# Create a play in a playbook with the python API
from ara.api.plays import PlayApi

data = {
    "playbook_id": 1,
    "name": "Play name",
    "started": "1970-08-14T00:52:49.570031"
}
play = PlayApi().post(data)
```

**For updating data**:

```
# Update a play in a playbook over http
curl -H "Content-Type: application/json" -X PATCH -d '{ "play_id": 1, "ended": "1970-08-14T00:52:49.570031" }' <ara>/api/v1/plays

# Update a play in a playbook with the python API
from ara.api.plays import PlayApi

data = {
    "play_id": 1,
    "ended": "1970-08-14T00:52:49.570031"
}
play = PlayApi().patch(data)
```

There might eventually be some "wrapper" methods for the Python API (i.e, "create_playbook")
but in the end, it will use the same API behind the scene.

## What's coming: input and output drivers

Currently, there is only one supported way to send data to ARA -- it's through the
Ansible callback.
However, some users have also requested different means of sending data to ARA.

An example that was given is that events could be dropped on a message bus (ex: mqtt, rabbitmq)
instead of being sent directly to the API or database and then ARA could pick up and
process the events asynchronously.

The input will be made modular and generic to allow the notion of "input drivers" so that
we can implement these new methods easily.

For output, in a similar fashion, it's currently possible to export data from ARA to
static HTML, junit or subunit.

There's also been interest in sending data to other backends such as graphite, elasticsearch or influxdb.

The appeal in using ARA to export data to these backends instead of using different Ansible callbacks
that already do that comes from having the data aggregated, organized and indexed in one place and
then being able to export the data asynchronously.

The current export methods will be folded back as "output drivers" and will use a common
interface to export data. We will be able to leverage this to make it easier to implement
other output drivers.

## What's coming: database improvements and backwards incompatibility

A significant amount of work has been done on the database model.
The magnitude of these changes have made me consider backwards incompatibility
for the first time since ARA introduced database upgrades in [ARA 0.10 from December 2016](https://github.com/openstack/ara/releases/tag/0.10.0).

The decision to move forward without supporting migrations and upgrades was to make this work possible and drastically easier.

I think the database layout is almost ready to go for 1.0, here's some examples of what has been done so far:

- Primary keys and identifiers are now integers instead of UUIDs
  (Thanks [Monty Taylor](http://lists.openstack.org/pipermail/openstack-dev/2017-April/115013.html) for the tip)
- The ``Stats`` and ``HostFacts`` tables were merged with the ``Hosts`` table
- The ``TaskResults`` table was renamed to ``Results`` because there's no other kind of results
- The ``Data`` table was renamed to ``Records`` because "Data" was an awful name and it better matches it's intended use case with the ``ara_record`` [module](http://ara.readthedocs.io/en/latest/usage.html#using-the-ara-record-module).
- Different fields have been removed or renamed to make their names more accurate and their meaning consistent throughout the application
- Some relationships between components were added, removed or changed

## What's coming: new features

All of the previous changes do not have much of an impact on end users. They are
all mostly under the hood and does not add much immediate value for users.

Here's some features I would like to land in 1.0:

- Add support for recording ad-hoc command execution (ex: ``ansible -m service -a "name=foo state=stopped``)
- Add support check/dry mode (``ansible-playbook --check``)
- Add support diff mode (``ansible-playbook --diff``)
- Add support for grouping or searching playbooks

**Edit**: The following features will also be landing in 1.0!

- Inventory, vars, host_vars and group_vars files will be saved with the report
- Role handlers, meta and defaults files will also be saved with the report
- Hosts will now display which groups they are a member of
- Playbook reports will be able to be named for displaying in the UI. For example
  you will be able to name a playbook "deploy mysql" and it will display that instead
  of the playbook file name "<playbook>.yml".

## When is all of that coming ?

So, when is all of that coming, you ask ?

Work is progressing very well on the API implementation and there should not be
too many changes remaining to do in the database model.

{{< tweet 897641159681593344 >}}

There has not been any work so far to refactor the backend to use the API and there
are also some features I'd like to land.

With all things considered, I would like to be able to release an alpha or a beta
release sometime during September. I'll reach out to ask users to test it out :)

If you'd like to follow the progress, I sometime post updates on Twitter, you can
follow me there as [@dmsimard](https://twitter.com/dmsimard).

Feel free to come hang around on IRC to chat about ARA as well !
You'll find users and myself in #ara on the Freenode server.

