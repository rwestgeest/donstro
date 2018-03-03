# Donstro

A simple utility to deploy docker containers remotely using docker-compose over ssh.

# usage

Donstro relies on having a set of configuration files in the donstro
directory in your app named after their configuration
(e.g. test or production). See more about this in the configuration
section below.

Given that you've taken care for that your would typically use:

* `donstro setup` to setup app

* `donstro up|rm|ps|stop|restart` to pass these tasks to docker-compose
  remotely  
  note that `up` is passed as `up -d` do make sure it is always detached

* `donstro rake <args>` to execute rake tasks in the container remotely
(provided you use **donstro** to deploy an application that uses rake to
perform build- and other tasks.

* `donstro do <args>` to execute **args** in the container.

All commands need to be run from your application's repo. By default
commands are run with STAGE=test and VERSION=$(cat VERSION) if you want
to override, run donstro with overrides of these environment variables

# installation

Clone the repo and put it in your path somewhere.

# configuration

In your app, you'll need a donstro directory for each stage. Suppose you
have test and production youll need:

~~~
donstro/
  test/
  production/
~~~

In each stage you'll need the following files:

~~~
donstro/test/
├── docker-compose.yml
├── donstro.env
└── remote.env
~~~

## docker-compose.yml
This file speaks for itself. It defines the service(s) you want to
deploy. It is not only used for deploying your services, but also for
running commands in a container with de same definition.

## donstro.env
This file should contain a few importand environment variables and is sourced in donstro.
The environment variables are essentials use to:

* access your server
* access the image definition in a docker registry (like docker hub)
* setup a remote donstro environment
* setup your app

### Accessing your server

~~~
USER=<the remote user>
HOST=<the remote host>
~~~

### access docker registry

~~~
DOCKER_REGISTRY=<the registry where your images are pushed>
DOCKER_IMAGE=<the image name>
~~~

Your docker image's version is taken from a VERSION file in your repo
or a VERSION environment variable, typically passed to donstro directly

### the remote donstro environment

~~~
APP_NAME=<your app name>
~~~

This could be the name of your service (if there's only one in your
docker-compose.yml), but it could be any other name. Here's how it is
used:

On your remote a dir, the following directory structure is created for
each `APP_NAME` and stage

~~~
donstro/
└── ${APP_NAME}/
    └── ${APP_NAME}_${STAGE}/
        └── docker-compose.yml
~~~

This makes sure that all apps stages are grouped by `APP_NAME`, 
services are named by `APP_NAME_STAGE_service` and possible volumes are
named `APP_NAME STAGE_volume_name`

### app setup

When running `donstro setup` the `docker-compose.yml` is copied to the
appropriate location (see above), and optionally the command specified
in the `SETUP_APP` variable is executed. For example in a rails app the
variable could look like this:

~~~
SETUP_APP="remote_compose run rake db:setup"
~~~

`remote_compose` In this case is a `donstro` function running a command
in a container as defined by the service name APP_NAME. It executes `rake
db:setup` to make sure the database is setup, migrated and seeded.

# the name

Naming things is hard. '*Donstro*' may not make any sense to you. Here's
how I came up with donstro.

As it is based on docker deployment, it is remotely related to docker.
Docker's uses a container metaphore and has a whale as an icon. Monstro
is the whale, that swallowed Gepetto in the famous Pinocchio tale.

Donstro is a mix of Docker and Monstro. Nothing more to it.
