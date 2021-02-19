---
kind: home
---

ARA Records Ansible and makes it easier to understand and troubleshoot.

It's another recursive acronym.

![ara-full-logo](/static/ara-full-logo.png)

## What it does

Simple to install and get started, ara provides reporting by saving detailed and
granular results of ``ansible`` and ``ansible-playbook`` commands wherever you run them:

- by hand or from a script
- from a laptop, a desktop, a container or a server
- for development, CI or production
- from a linux distribution or even on OS X (as long as you have ``python >= 3.5``)
- from tools such as AWX or Tower, Jenkins, GitLab CI, Rundeck, Zuul, Molecule, ansible-pull, ansible-test or ansible-runner

By default, ara's Ansible callback plugin will record data to a local sqlite
database without requiring you to run a server or a service:

![quickstart-default](/static/ara-quickstart-default.gif)

ara can also provide a single pane of glass when recording data from multiple
locations by pointing the callback plugin to a running API server:

![quickstart-server](/static/ara-quickstart-server.gif)

The data is then made available for browsing, searching and querying over the
included reporting interface, a CLI client as well as a REST API.

## How it works

ARA Records Ansible execution results to sqlite, mysql or postgresql databases
by using an [Ansible callback plugin](https://docs.ansible.com/ansible/latest/plugins/callback.html).

This callback plugin leverages built-in python API clients to send data to a
REST API server:

![recording-workflow](/static/recording-workflow.png)

## What it looks like

### API browser

Included by the API server with django-rest-framework, the API browser allows
users to navigate the different API endpoints and query recorded data.

![ui-api-browser](/static/ui-api-browser.png)

### Reporting interface

A simple reporting interface built-in to the API server without any extra dependencies.

![ui-playbook-details](/static/ui-playbook-details.png)

### ara CLI

A built-in CLI client for querying and managing playbooks and their recorded data.

![cli-playbook-list](/static/cli-playbook-list.png)

The full list of commands, their arguments as well as examples can be found in
the [CLI documentation](https://ara.readthedocs.io/en/latest/cli.html#cli-ara-api-client).

## Getting started

### Requirements

- Any recent Linux distribution or Mac OS with python >=3.5 available
- The ara Ansible plugins must be installed for the same python interpreter as Ansible itself

For RHEL 7 and CentOS 7 it is recommended to run the API server in a container due to missing or outdated dependencies.
See this [issue](https://github.com/ansible-community/ara/issues/99) for more information.

### Recording playbooks without an API server

With defaults and using a local sqlite database:

```bash
# Install Ansible and ARA (with API server dependencies) for the current user
python3 -m pip install --user ansible "ara[server]"

# Configure Ansible to use the ARA callback plugin
export ANSIBLE_CALLBACK_PLUGINS="$(python3 -m ara.setup.callback_plugins)"

# Run an Ansible playbook
ansible-playbook playbook.yaml

# Use the CLI to see recorded playbooks
ara playbook list

# Start the built-in development server to browse recorded results
ara-manage runserver
```

### Recording playbooks with an API server

You can get an API server deployed using the [ara Ansible collection](https://github.com/ansible-community/ara-collection)
or get started quickly using the container images from [DockerHub](https://hub.docker.com/r/recordsansible/ara-api) and
[quay.io](https://quay.io/repository/recordsansible/ara-api):

```bash
# Create a directory for a volume to store settings and a sqlite database
mkdir -p ~/.ara/server

# Start an API server with podman from the image on DockerHub:
podman run --name api-server --detach --tty \
    --volume ~/.ara/server:/opt/ara:z -p 8000:8000 \
    docker.io/recordsansible/ara-api:latest

# or with docker from the image on quay.io:
docker run --name api-server --detach --tty \
    --volume ~/.ara/server:/opt/ara:z -p 8000:8000 \
    quay.io/recordsansible/ara-api:latest
```

Once the server is running, ara's Ansible callback plugin must be installed and configured to send data to it:

```bash
# Install Ansible and ARA (without API server dependencies) for the current user
python3 -m pip install --user ansible ara

# Configure Ansible to use the ARA callback plugin
export ANSIBLE_CALLBACK_PLUGINS="$(python3 -m ara.setup.callback_plugins)"

# Set up the ARA callback to know where the API server is located
export ARA_API_CLIENT="http"
export ARA_API_SERVER="http://127.0.0.1:8000"

# Run an Ansible playbook
ansible-playbook playbook.yaml

# Use the CLI to see recorded playbooks
ara playbook list
```

Data will be available on the API server in real time as the playbook progresses and completes.

You can read more about how container images are built and how to run them in the
[documentation](https://ara.readthedocs.io/en/latest/container-images.html).