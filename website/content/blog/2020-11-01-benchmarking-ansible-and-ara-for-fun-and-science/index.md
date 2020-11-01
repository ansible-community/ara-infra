---
author: "David Moreau Simard"
categories:
  - experiments
tags:
  - ansible
date: 2020-11-01
title: "Benchmarking ansible and ara for fun and science"
slug: benchmarking-ansible-and-ara-for-fun-and-science
type: post
---

We've had issues opened about benchmarking and improving the performance of the
[ara callback plugin](https://github.com/ansible-community/ara/issues/171) as
well as the [API server](https://github.com/ansible-community/ara/issues/170) for
a while now and I only recently took a bit of time to tackle the callback.

It paid off because we can already see significant performance benefits in
the 1.5.3 release of ara and it builds a foundation for future improvement
opportunities.

If you'd like to see the raw unformatted data that was used for this post, check
out this [gist on GitHub](https://gist.github.com/dmsimard/ab6d53ac2edf3b4ab6076e20dcad8fe9).

## A benchmarking playbook

Whenever you want to improve something, it's important to measure it first so you
know how better (or worse!) things become as a result of your changes.

The first step was to create a standardized benchmarking playbook that we could
run across a variety of configurations and parameters.

The playbook was designed to run a specified number of tasks against a specified
number of hosts.

It's available in the [git repository](https://github.com/ansible-community/ara/blob/master/tests/integration/benchmark.yaml)
but it's simple and small enough to include here:

```yaml
# Copyright (c) 2020 The ARA Records Ansible authors
# GNU General Public License v3.0+ (see COPYING or https://www.gnu.org/licenses/gpl-3.0.txt)

- name: Create many hosts
  hosts: localhost
  gather_facts: no
  vars:
    benchmark_host_count: 25
  tasks:
    - name: Add a host to the inventory
      add_host:
        ansible_connection: local
        hostname: "host-{{ item }}"
        groups: benchmark
      with_sequence: start=1 end={{ benchmark_host_count }}

- name: Run tasks on many hosts
  hosts: benchmark
  vars:
    benchmark_task_file: "{{ playbook_dir }}/benchmark_tasks.yaml"
    # Run N tasks per host
    benchmark_task_count: 50
    # Off by default to prevent accidental load spike on localhost
    benchmark_gather_facts: no
  gather_facts: "{{ benchmark_gather_facts }}"
  tasks:
    - name: Include a task file
      include_tasks: "{{ benchmark_task_file }}"
      with_sequence: start=1 end={{ benchmark_task_count }}
```

and then the [benchmark_tasks.yaml](https://github.com/ansible-community/ara/blob/master/tests/integration/benchmark_tasks.yaml) file:

```yaml
# Copyright (c) 2020 The ARA Records Ansible authors
# GNU General Public License v3.0+ (see COPYING or https://www.gnu.org/licenses/gpl-3.0.txt)

# These are tasks meant to be imported by benchmark.yaml

- name: Run a task
  debug:
    msg: "{{ inventory_hostname }} running task {{ item }}/{{ benchmark_task_count }}"
```

## Methodology

All tests ran under Ansible 2.10.2 and ``ANSIBLE_FORKS=50`` using the default sqlite database backend for ara.

The benchmark playbook was run three times:

```bash
# 25 hosts and 50 tasks: 1276 results
ansible-playbook -i 'localhost,' -c local tests/integration/benchmark.yaml

# 100 hosts and 50 tasks: 5101 results
ansible-playbook -i 'localhost,' -c local tests/integration/benchmark.yaml \
    -e benchmark_host_count=100

# 200 hosts and 200 tasks: 40201 results
ansible-playbook -i 'localhost,' -c local tests/integration/benchmark.yaml \
    -e benchmark_host_count=200 \
    -e benchmark_task_count=200
```

*Note: localhost and two bootstrap tasks are included in the results below*

## ansible without ara

| tasks | hosts | results | duration |
|-------|-------|---------|----------|
|    52 |    26 |    1276 |   0m 11s |
|    52 |   101 |    5101 |   0m 41s |
|   202 |   201 |   40201 |   6m 01s |

Our control: basically how much time these playbooks take to run without ara enabled so we can calculate the overhead
and performance when enabling the ara callback.

## ansible with ara 1.5.1

| api client | api server | tasks | hosts | results | duration |
|------------|------------|-------|-------|---------|----------|
|  offline   |   django   |    52 |    26 |    1276 |   0m 56s |
|  http      |   django   |    52 |    26 |    1276 |   0m 32s |
|  http      |  gunicorn  |    52 |    26 |    1276 |   0m 36s |
|  offline   |   django   |    52 |   101 |    5101 |   3m 31s |
|  http      |   django   |    52 |   101 |    5101 |   1m 39s |
|  http      |  gunicorn  |    52 |   101 |    5101 |   2m 19s |
|  offline   |   django   |   202 |   201 |   40201 |   30m22s |
|  http      |   django   |   202 |   201 |   40201 |   17m28s |
|  http      |  gunicorn  |   202 |   201 |   40201 |   21m38s |

1.5.1 is the latest version that didn't implement threading inside the callback plugin.

It's curious that the django built-in webserver outperformed running with gunicorn when using the http client.
I was not able to reproduce this result in 1.5.3.

I was aware that there was an overhead when enabling the callback but never realized
the performance hit was this much until now, taking the time to accurately measure it:

| results | without ara |  1.5.1 | overhead |
|---------|-------------|--------|----------|
|    1276 |         11s |    32s |      21s |
|    5101 |         41s |  1m39s |      58s |
|   40201 |       6m01s | 17m28s |   ~11.4m |

## ansible with ara 1.5.3

| api client | api server | tasks | hosts | results | duration |
|------------|------------|-------|-------|---------|----------|
|  offline   |   django   |    52 |    26 |    1276 |   0m 52s |
|  http      |   django   |    52 |    26 |    1276 |   0m 30s |
|  http      |  gunicorn  |    52 |    26 |    1276 |   0m 20s |
|  offline   |   django   |    52 |   101 |    5101 |   3m 22s |
|  http      |   django   |    52 |   101 |    5101 |   1m 37s |
|  http      |  gunicorn  |    52 |   101 |    5101 |   1m 09s |
|  offline   |   django   |   202 |   201 |   40201 |   29m25s |
|  http      |   django   |   202 |   201 |   40201 |   17m24s |
|  http      |  gunicorn  |   202 |   201 |   40201 |   13m47s |

1.5.2 introduced threading in the callback and then 1.5.3 was subsequently released to workaround
an [issue when using the offline client](https://github.com/ansible-community/ara/issues/183), forcing it to use a single thread for now.

From the table above, we can tell:

- Running a single thread with the offline client is just about the same performance as 1.5.1 without threading, if only a little bit faster.
- There is a significant improvement in performance due to the multi-threading when using the http client
- Running the API server with gunicorn outperforms using the built-in django development server

1.5.3 reduced the overhead of the callback plugin quite a bit when comparing to 1.5.1:

| results | without ara |  1.5.3 | overhead |
|---------|-------------|--------|----------|
|    1276 |         11s |    20s |       9s |
|    5101 |         41s | 1m 09s |      28s |
|   40201 |      6m 01s | 13m47s |    ~7.8m |

## For science: ara 0.16.8

| tasks | hosts | results | duration |
|-------|-------|---------|----------|
|    52 |    26 |    1276 |   0m 28s |
|    52 |   101 |    5101 |   1m 56s |
|   202 |   201 |   40201 |   19m05s |

Although ara 0.x is no longer supported, it turns out it still works if you use the [stable/0.x](https://github.com/ansible-community/ara/tree/stable/0.x) git branch.. even with Ansible 2.10!

It was interesting to run the same benchmark against 0.x because it runs a completely different backend.
It uses flask instead of django and doesn't provide an API: the callback talks directly to the database through flask-sqlalchemy.

## Putting it all together

Tallying up the numbers, we can see that we're on the right track and performance is improving:

| tasks | hosts | results | without ara | 0.16.8 |   1.5.1 |   1.5.3 |
|-------|-------|---------|-------------|--------|---------|---------|
|    52 |    26 |    1276 |         11s |    28s |     32s |     20s |
|    52 |   101 |    5101 |         41s | 1m 56s |  1m 39s |  1m 09s |
|   202 |   201 |   40201 |      6m 01s | 19m05s |  17m28s |  13m47s |

There is definitely more work to do and more opportunities to improve performance to find.
There will unfortunately always be an overhead but it needs to be low enough that it's worth it without sacrificing simplicity.

In the future, it could be interesting the measure the impact of other parameters on performance like:

- Ansible forks -- what difference does having 25, 100 or 200 forks make ?
- Callback threads -- is there a benefit when running more threads in the threadpool ?
- Version of Python -- is there any difference between python 3.5 and 3.9 ?
- Version of Ansible -- was there any performance improvements or regressions between 2.8 and 2.10 ?
- Database backend -- is sqlite faster than mysql ? what about postgresql ?
- Application backend -- is gunicorn faster than uwsgi ? what about apache mod_wsgi ?
- Latency: what's the impact on performance of adding a jump box ? what about 50ms ? 250ms ?

If you'd like to help, have a look at the issues on GitHub or come chat with us
on [Slack or IRC](https://ara.recordsansible.org/community/) !

See you around o/
