
# Getting Started with Docker
-----------------------------

**SGV Linux Users Group Meetup**

| Dan Isla
| Data Scientist at JPL
| June 11, 2015

Running This Presentation
=========================

Start `shellinabox <https://code.google.com/p/shellinabox/>`__

::

    $ brew install shellinabox
    $ sudo shellinaboxd -s/:LOGIN -t --no-beep

Create the reveal.js html

::

    $ ipython nbconvert docker-talk.ipynb --to slides

Start the webserver with proxy to ``shellinaboxd`` at ``/shell``

::

    $ python tape.py -p 8000 -P /shell=http://localhost:4200

--------------

#### shellinabox
----------------

.. raw:: html

   <p>
   <iframe class="shellinabox" src="http://localhost:8000/shell/" frameborder="1" width="100%" height="100%"></iframe>
   </p>

# About Me
----------

What is Docker?
===============

Trends
------

.. raw:: html

   <p align="center">
     

.. raw:: html

   </p>

## Containers vs VMs
--------------------



## Docker Architecture
----------------------

 https://docs.docker.com/introduction/understanding-docker/

# Docker makes Linux containers easy
------------------------------------

-  Written in Go
-  Manages the creation of kernel-level isolation between processes
-  Manages the network interfacing between the Host OS
-  Manages the storage interface between the Host OS
-  Uses a copy-on-write filesystem to create shared layers in images,
   similar to Git
-  Images can be built and shipped to any VM running the Docker daemon.

# Installing Docker
-------------------

Official Install guides:
https://docs.docker.com/installation/#installation

Basic requirements:

-  64-bit Linux
-  Kernel 3.10

### Linux
---------

Latest stable version available in your favorite package manager.

OR the quick and dirty method:

::

    $ wget -qO- https://get.docker.com/ | sh

### OSX
-------

OSX is built on Unix, not Linux and doesn't have the required kernel
features.

Some options: - boot2docker (my preferred) - Kitematic (graphical
interface) - Vagrant (great for multi-node)

### Docker host under Linux vs OSX
----------------------------------

# Container Hello World
-----------------------

Pulling your first container
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

::

    $ docker pull ubuntu

Running your first command
^^^^^^^^^^^^^^^^^^^^^^^^^^

::

    $ docker run ubuntu echo hello world

## Container Environment
------------------------

.. raw:: html

   <p>
   <iframe class="shellinabox" src="http://localhost:8000/shell/" frameborder="1" width="100%" height="400px"></iframe>
   </p>

# Simple Web Service Example
----------------------------

`Twelve-Factor <12factor.net>`__ All the Things!
------------------------------------------------

| **Processes**
| Execute the app as one or more stateless processes

| **Config**
| Store config in the environment

| **Dependencies**
| Explicitly declare and isolate dependencies

| **Build, release, run**
| Strictly separate build and run stages

| **Disposability**
| Maximize robustness with fast startup and graceful shutdown

| **Dev/prod parity**
| Keep development, staging, and production as similar as possible

## Dockerfile
-------------

::

    FROM python:2.7

    WORKDIR /opt/app

    EXPOSE 8000

    ENTRYPOINT python -m SimpleHTTPServer

Build the image
~~~~~~~~~~~~~~~

::

    $ docker build -t python-webserver .

Run the container in the foreground
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

::

    $ docker run -it -P python-webserver

## Demo
-------

.. raw:: html

   <p>
   <iframe class="shellinabox" src="http://localhost:8000/shell/" frameborder="1" width="100%" height="400px"></iframe>
   </p>

### Data Volumes
----------------

Bind-mounting data volume to the container

::

    $ docker run -it --rm -p 8001:8000 \
        -v ~/Projects/ipython/docker-talk/examples/python-webserver:/opt/app \
        python-webserver

New args: - ``--rm``: remove container after exiting - ``-p 8001:8000``:
Map docker host port ``8001`` to *container* port ``8000`` - ``-v``:
Bind mount my local ``python-webserver`` directory to the ``/opt/app``
directory inside the *container*.

# Rails app Example
-------------------

Ruby typically has a ton of dependencies and can be a pain to setup a
dev environment and and deploy.

Read about the official Rails docker image:
https://registry.hub.docker.com/u/library/rails/

Pull in the ``rails:4`` and ``rails:onbuild`` image:

::

    $ docker pull rails:4
    $ docker pull rails:onbuild

# Build the rails container
---------------------------

The ``onbuild`` base image makes it very simple to bootstrap the
environment with a 1-line Dockerfile.

Dockerfile:

::

    FROM rails:onbuild

Per the rails image Docker Hub description:

    This image includes multiple ``ONBUILD`` triggers which should cover
    most applications. The build will ``COPY . /usr/src/app``,
    ``RUN bundle install``, ``EXPOSE 3000``, and set the default command
    to ``rails server``.

Example Rails app from GitHub: https://github.com/engineyard/todo

### Lets try it
---------------

.. raw:: html

   <p>
   <iframe class="shellinabox" src="http://localhost:8000/shell/" frameborder="1" width="100%" height="400px"></iframe>
   </p>

Clone the repo:

::

    $ git clone https://github.com/engineyard/todo

Make the Dockerfile:

::

    $ echo "FROM rails:onbuild" > todo/Dockerfile

Build the image:

::

    $ cd todo
    $ docker build -t rails-todo .

Run the container:

::

    $ docker run -it --rm -p 3000:3000 -e RAILS_ENV=production rails-todo

Open in browser at: http://dockerhost:3000

Oops! We need to migrate the database, lets bind mount the sqlite3 file
and rake migrate

::

    $ touch db/development.sqlite3
    $ docker run -it --rm -p 3000:3000 \
        -e RAILS_ENV=development \
        -v ~/Projects/ipython/docker-talk/examples/rails-app/todo/db/development.sqlite3:/usr/src/app/db/development.sqlite3:rw \
        rails-todo bin/rake db:migrate

Now run it in the background (detached mode):

::

    $ docker run -d \
        --name rails-todo \
        -p 3000:3000 \
        -e RAILS_ENV=development \
        -v ~/Projects/ipython/docker-talk/examples/rails-app/todo/db/development.sqlite3:/usr/src/app/db/development.sqlite3:rw \
        rails-todo

Tail the output:

::

    $ docker logs -f rails-todo

# Go Example
------------

::

    $ docker build -t go-api .
    $ docker run -it --rm -p 8080:8080 go-api

# Demo
------

.. raw:: html

   <p>
   <iframe class="shellinabox" src="http://localhost:8000/shell/" frameborder="1" width="100%" height="400px"></iframe>
   </p>

# NodeJS Service Example
------------------------

Dockerfile:
~~~~~~~~~~~

::

    FROM node:0.10.38-slim

    ADD package.json /tmp/package.json
    RUN cd /tmp && npm install
    RUN mkdir -p /opt/app && cp -a /tmp/node_modules /opt/app/

    WORKDIR /opt/app

    EXPOSE 8080

    ADD server.js package.json /opt/app/

    ENTRYPOINT node server.js

Caching
~~~~~~~

On subsequent builds, docker will only execute the ``npm install`` when
``package.json`` changes. A hash of the successful command execution is
saved so anything prior the last changed line is skipped when building.

# Demo
------

.. raw:: html

   <p>
   <iframe class="shellinabox" src="http://localhost:8000/shell/" frameborder="1" width="100%" height="400px"></iframe>
   </p>

# Prototyping Example
---------------------

Docker is a great tool for trying to technologies
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

-  Elasticsearch
-  Hadoop
-  Spark

# Composing Containers
----------------------

docker-compose.yml
~~~~~~~~~~~~~~~~~~

::

    elasticsearch:
      image: deviantony/elk-elasticsearch
      ports:
        - "9200:9200"
    kibana4:
      image: deviantony/elk-kibana
      ports:
        - "5601:5601"
      links:
        - elasticsearch

Starting:

::

    $ docker-compose up

https://github.com/deviantony/fig-elk/blob/master/docker-compose.yml

# Demo
------

.. raw:: html

   <p>
   <iframe class="shellinabox" src="http://localhost:8000/shell/" frameborder="1" width="100%" height="400px"></iframe>
   </p>

# Legacy app containers (coffins)
---------------------------------

## Example Journey to the Graveyard - Trac
------------------------------------------

Legacy Stack:
^^^^^^^^^^^^^

::

              ┌───────────────────────────┐
              | Institutional LDAP Server | 
              └────────────┬──────────────┘
                           X
    ┌──────────────────────┴────────────────────────┐
    | Apache with LDAP, SSL and WebDAV and Trac CGI | 
    ├───────────────────────────────────────────────┤
    │                   RHEL 4.3                    │
    ├───────────────────────────────────────────────┤
    │            Bare metal or 32bit VM             │
    └───────────────────────────────────────────────┘

Goal:
^^^^^

::

                   ┌───────────────────────────┐
                   | Institutional LDAP Server | 
                   └────────────┬──────────────┘
                                |
    ┌───────────────────────┬───┴─────────────────┬──────┐
    | Local Docker Registry | Apache w/SVN WebDAV | Trac | 
    ├───────────────────────┴─────────────────────┴──────┤
    │                         Docker                     │
    ├────────────────────────────────────────────────────┤
    │                        CentOS 7                    │
    ├────────────────────────────────────────────────────┤
    │                  GovCloud EC2 64bit VM             │
    └────────────────────────────────────────────────────┘

# Security with SELinux
-----------------------

Process isolation is great but what about storage and networking? SELinux!!
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Applying container label to filesystem:

::

    /usr/bin/chcon -Rt svirt_sandbox_file_t /data

Now any container can bind mount the host os ``/data`` directory and
safely have read-write access.

SELinux networking policies could also be used to prevent compromised
containers from being used as springboards into your network.

# Deploying Containers
----------------------

TODO: - Show ops-dev meltdown meme - Docker push to hub - Talk about
github integration with Docker Hub - Talk about private registry - Show
systemd example

Running with ``systemd`` and a local repository
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

::

    [Unit]
    Description=Docker Registry
    Requires=docker-registry.service
    After=docker-registry.service

    [Service]
    Restart=on-failure
    TimeoutStartSec=0
    ExecStartPre=-/usr/bin/docker kill registry
    ExecStartPre=-/usr/bin/docker rm registry
    ExecStartPre=/usr/bin/docker pull registry:0.9.1
    ExecStart=/usr/bin/docker run --rm\
      --name=registry\
      --publish="5000:5000"\
      --env="SETTINGS_FLAVOR=s3"\
      --env="AWS_REGION=us-west-2"\
      --env="AWS_ENCRYPT=true"\
      --env="AWS_SECURE=true"\
      --env="AWS_BUCKET=exmaple-com-docker-images"\
      --env="STORAGE_PATH=/registry"\
      --env="GUNICORN_OPTS=[--preload]"\
      --env="GUNICORN_WORKERS=4"\
      --env="SEARCH_BACKEND=sqlalchemy"\
      registry:0.9.1
    ExecStop=-/usr/bin/docker stop registry

    [Install]
    WantedBy=multi-user.target

Apache vhost service
~~~~~~~~~~~~~~~~~~~~

::

    [Unit]
    Description=Apache SSL LDAP SVN
    Requires=docker-registry.service
    After=docker-registry.service

    [Service]
    Restart=on-failure
    TimeoutStartSec=0
    ExecStartPre=-/usr/bin/chcon -Rt svirt_sandbox_file_t /data/httpd
    ExecStartPre=-/usr/bin/chcon -Rt svirt_sandbox_file_t /data/trac-eol-repo
    ExecStartPre=-/usr/bin/docker kill apache
    ExecStartPre=-/usr/bin/docker rm apache
    ExecStartPre=/usr/bin/docker pull localhost:5000/apache-ssl-ldap-svn:1.0.0-2
    ExecStart=/usr/bin/docker run --rm\
      --name=apache\
      --net=host\
      --volume="/data/httpd/logs:/var/log/httpd:rw"\
      --volume="/data/httpd/ssl/server.key:/etc/httpd/ssl/server.key:ro"\
      --volume="/data/httpd/ssl/server.crt:/etc/httpd/ssl/server.crt:ro"\
      --volume="/data/webapp-repo/deploy/httpd:/etc/httpd/conf.d"\
      --volume="/data/httpd/html:/var/www/html:ro"\
      localhost:5000/apache-ssl-ldap-svn:1.0.0-2
    ExecStop=-/usr/bin/docker stop apache
    ExecReload=/usr/bin/docker exec apache apachectl -k graceful

    [Install]
    WantedBy=multi-user.target

# Container Orchestration
-------------------------

Questions ?
===========

-  dan.isla@gmail.com
    https://github.com/danisla

Convert Notebook to RST
-----------------------

.. code:: python

    %%bash
    ipython nbconvert docker-talk.ipynb --to rst


.. parsed-literal::

    [NbConvertApp] Converting notebook docker-talk.ipynb to rst
    [NbConvertApp] Writing 882 bytes to docker-talk.rst


Convert Notebook and Start Slide Show
-------------------------------------

.. code:: python

    %%bash
    
    function cleanup {
        # Cleanup any old revealjs server
        kill `pgrep -f "python nbconvert"` 2>/dev/null
    }
    trap cleanup EXIT INT
    
    # Start revealjs server with explict host and port (Docker friendly)
    ipython nbconvert \
        --ServePostProcessor.open_in_browser=False \
        --ServePostProcessor.ip=0.0.0.0 \
        --ServePostProcessor.port=8000 \
        docker-talk.ipynb --to slides --post serve


.. parsed-literal::

    Process is terminated.


Convert Notebook to Reveal.js slides
------------------------------------

.. code:: python

    %%bash
    ipython nbconvert docker-talk.ipynb --to slides


.. parsed-literal::

    [NbConvertApp] Converting notebook docker-talk.ipynb to slides
    [NbConvertApp] Writing 229940 bytes to docker-talk.slides.html

