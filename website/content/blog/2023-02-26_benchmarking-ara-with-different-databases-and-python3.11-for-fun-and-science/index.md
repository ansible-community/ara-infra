---
author: "David Moreau Simard"
categories:
  - experiments
tags:
  - ansible
date: 2023-02-26
title: "Benchmarking ara with different databases and python 3.11 for fun and science"
slug: benchmarking-ara-with-different-databases-and-python3.11-for-fun-and-science
type: post
---

We've last [benchmarked ara itself](https://ara.recordsansible.org/blog/2020/11/01/benchmarking-ansible-and-ara-for-fun-and-science/) as well as [different versions of python and ansible](https://ara.recordsansible.org/blog/2021/01/30/benchmarking-ansible-and-python-versions-for-fun-and-science/) but it's been a while !

Since then:

- ansible-core 2.11, 2.12, 2.13 and 2.14 have released (time flies!)
- python 3.10 as well as 3.11 have come out
- ara has had a few releases with the latest being 1.6.1

Python 3.11 is reported to be [up to 10-60% faster than Python 3.10](https://www.python.org/downloads/release/python-3110b3/) and it felt like a good opportunity for some new benchmarking metrics including database backends.

This time we will gather data about:
- ansible-core 2.11.12 and 2.14.2 under both python3.9.16 and python3.11.2, with and without ara
- the sqlite, mysql and postgresql django 3.2.18 database backends for the ara API server

We'll use [gunicorn](https://gunicorn.org/) in every test and leave the benchmarking of web and application servers for another day.

Using a similar methodology as before, we will run a synthetic benchmark (written as an [ansible playbook](https://github.com/ansible-community/ara/blob/master/tests/integration/benchmark.yaml)) just to measure the difference between versions and database backends.

Feel free to skip to the end for the numbers but if you're interested in the setup and how we'll run the benchmark, read on !

## The python and ansible setup

We'll run our non-scientific tests in my basement's [OpenStack](https://www.openstack.org/) cloud running on a single server with modest hardware from 2012.

Our virtual machine runs a Fedora 38 [beta cloud image](https://openqa.fedoraproject.org/nightlies.html) with 16 cores, 32GB of RAM backed by local SSDs.

Fedora 38 conveniently ships with python3.11 by default and includes a python3.9 package that we can install along with our development dependencies:
```bash
dnf install -y gcc \
  python3.11-devel \
  python3.9 \
  python3.9-devel \
  postgresql \
  libpq \
  libpq-devel \
  mariadb-connector-c \
  mariadb-connector-c-devel \
```

Now we create python virtual environments in which we'll install our first version of ansible-core and gunicorn to run the ara server:
```
python3.9 -m venv ~/venv-python3.9
~/venv-python3.9/bin/pip install 'ansible-core<2.12' ara[server,mysql,postgresql] gunicorn

python3.11 -m venv ~/venv-python3.11
~/venv-python3.11/bin/pip install 'ansible-core<2.12' ara[server,mysql,postgresql] gunicorn
```

Once we've run all the tests with ansible-core 2.11.12, we'll upgrade it to 2.14.2 before running every test again.

## The database setup

We'll install podman to run local MariaDB and PostgreSQL servers with default settings:
```bash
dnf -y install podman
podman run -d \
  --name mariadb \
  --net=host \
  -e MYSQL_DATABASE=ara \
  -e MYSQL_USER=ara \
  -e MYSQL_PASSWORD=password \
  docker.io/mariadb:10
podman run -d \
  --name postgresql \
  --net=host \
  -e POSTGRES_DB=ara \
  -e POSTGRES_USER=ara \
  -e POSTGRES_PASSWORD=password \
  docker.io/postgres:10
```

Between each test, we'll switch between database backends using a symbolic link between three ``~/.ara/server/settings.yaml`` files with the following:
```
settings.yaml_mysql:  DATABASE_CONN_MAX_AGE: 60
settings.yaml_mysql:  DATABASE_ENGINE: django.db.backends.mysql
settings.yaml_mysql:  DATABASE_HOST: 127.0.0.1
settings.yaml_mysql:  DATABASE_NAME: ara
settings.yaml_mysql:  DATABASE_OPTIONS: {}
settings.yaml_mysql:  DATABASE_PASSWORD: password
settings.yaml_mysql:  DATABASE_PORT: 3306
settings.yaml_mysql:  DATABASE_USER: ara

settings.yaml_postgresql:  DATABASE_CONN_MAX_AGE: 60
settings.yaml_postgresql:  DATABASE_ENGINE: django.db.backends.postgresql
settings.yaml_postgresql:  DATABASE_HOST: 127.0.0.1
settings.yaml_postgresql:  DATABASE_NAME: ara
settings.yaml_postgresql:  DATABASE_OPTIONS: {}
settings.yaml_postgresql:  DATABASE_PASSWORD: password
settings.yaml_postgresql:  DATABASE_PORT: 5432
settings.yaml_postgresql:  DATABASE_USER: ara

settings.yaml_sqlite:  DATABASE_CONN_MAX_AGE: 0
settings.yaml_sqlite:  DATABASE_ENGINE: django.db.backends.sqlite3
settings.yaml_sqlite:  DATABASE_HOST: null
settings.yaml_sqlite:  DATABASE_NAME: /root/.ara/server/ansible.sqlite
settings.yaml_sqlite:  DATABASE_OPTIONS: {}
settings.yaml_sqlite:  DATABASE_PASSWORD: null
settings.yaml_sqlite:  DATABASE_PORT: null
settings.yaml_sqlite:  DATABASE_USER: null
```

The default ara API client, dubbed ``offline``, takes care of running SQL migrations automatically so you don't need to: this is why all you need to do to get started is to install ara and enable the callback plugin.

However, for these tests, we will be running a server with gunicorn and we'll use ``ARA_API_CLIENT=http`` to enable the callback's threading with ``ARA_CALLBACK_THREADS=4`` when using the MySQL and PostgreSQL backends.
This will allow us to get the best possible performance by multi-threading things that can be and letting Ansible's playbook continue to run without waiting for the callback's every request to return.

In the case of sqlite we will stay single threaded since the database backend is prone to table lock concurrency errors when recording multiple playbooks at the same time or when multi-threading.

So, anyway, before running our tests we'll first need to initialize each database backend once by running SQL migrations with ``ara-manage migrate``.

## Running the ara server with gunicorn

We'll run the latest version of gunicorn as of writing (20.1.0) and we will switch between python 3.9 and 3.11 along with ansible-core and ara so it may also come into play to some extent when considering performance improvements.

We should probably run our ara server with the [recommended number of gunicorn workers](https://docs.gunicorn.org/en/stable/design.html#how-many-workers) but we'll use a bit less to leave room for ansible-core itself as well as the databases:

``gunicorn --workers=16 --access-logfile - --bind 0.0.0.0:8000 ara.server.wsgi``

## Running the benchmarking playbook

Running our benchmarking playbook looks like this:

```bash
git clone https://github.com/ansible-community/ara src/ara
export ANSIBLE_FORKS=50
time ~/venv-python3.9/bin/ansible-playbook src/ara/tests/integration/benchmark.yaml \
  -e benchmark_host_count=100 \
  -e benchmark_task_count=100
```

This gets us our "control" playbook with 100 hosts each running 100 tasks leading to ~10000 individual results without ara enabled.

When enabling ara, we'll export the following, as necessary, before running the same playbook:

```bash
export ARA_API_CLIENT=http
# export ARA_API_SERVER=http://127.0.0.1:8000  <-- this is already the default value

export ANSIBLE_CALLBACK_PLUGINS=$(~/venv-python3.9/bin/python3 -m ara.setup.callback_plugins)
# or export ANSIBLE_CALLBACK_PLUGINS=$(~/venv-python3.11/bin/python3 -m ara.setup.callback_plugins)

# when using mysql and postgresql:
export ARA_CALLBACK_THREADS=4
```

## The data and takeaways

After running running all 16 tests, we come up with the following data:

|                            | without ara | ara & sqlite | ara & mysql | ara & postgresql |
|----------------------------|-------------|--------------|-------------|------------------|
| ansible-core 2.11 & py3.9  | 5m7.421s    | 9m12.214s    | 7m3.097s    | 7m8.866s         |
| ansible-core 2.11 & py3.11 | 4m54.570s   | 8m38.533s    | 6m44.242s   | 6m48.962s        |
| ansible-core 2.14 & py3.9  | 4m48.924s   | 8m53.098s    | 7m7.989s    | 7m11.829s        |
| ansible-core 2.14 & py3.11 | 4m38.665s   | 8m26.335s    | 6m49.418s   | 6m46.373s        |

It is expected that running Ansible without ara will yield better performance because Ansible isn't busy letting ara record everything before moving on to the next task -- this is [explained in the docs](https://ara.readthedocs.io/en/latest/troubleshooting.html#improving-playbook-recording-performance).

With everything else being equal, we see a positive trend of performance improvements for both python and ansible-core:

- ansible-core 2.14 was up to ~20s faster than 2.11 for running 100 tasks across 100 hosts
- python 3.11 was up to ~34s faster than 3.9 when including ara and gunicorn into the mix

The django database backends for MySQL and PostgreSQL had quite similar results.

Granted, we did not attempt any particular performance tuning here and maybe we can try that some day but in the meantime it means you should probably pick what you already have or your favorite option if you wanted to deploy it in production.

Another thing to be mindful of is that, for our tests, the database server was local which meant little to no network latency so this could be an entirely different story if your database server is hosted halfway across the world.

Meanwhile, the lesser performance with sqlite here is mostly attributable to the lack of background threading in order to avoid table lock contention errors.
I use it in a lot of use cases and it is often good enough with sqlite's advantages of portability without being impacted by network latency or needing to run a server.

If you want to consider using sqlite at a large scale, the [distributed sqlite backend](https://ara.readthedocs.io/en/latest/distributed-sqlite-backend.html) was designed to [record over a million playbooks per month](https://ara.recordsansible.org/blog/scaling-ara-to-a-million-ansible-playbooks-a-month/) so it could be a good fit for your use case. It is a different approach that shards results in individual databases while still giving you a central server to browse results.

## In conclusion

In any case, from a performance perspective, it seems like upgrading your setup to the latest version of python and ansible-core is worth it, especially if you're running long playbooks against a large inventory.

Maybe next time we'll look at performance for application servers like gunicorn or uwsgi and web servers like apache or nginx.

By then, perhaps we will have a working implementation of [prometheus exporters for django and playbook metrics](https://github.com/ansible-community/ara/issues/177) to get us detailed information and pretty graphs !

Until then, stay up to date with the project or come chat with us:

- On Mastodon: https://fosstodon.org/@ara
- On IRC, Slack or Matrix: https://ara.recordsansible.org/community/

o/
