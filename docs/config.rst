.. _config:

=============
Configuration
=============

The blockade configuration file is conventionally named ``blockade.yaml`` and
is used to describe the containers in your application. Here is an example:

.. code-block:: yaml

    containers:
      c1:
        image: my_docker_image
        command: /bin/myapp
        volumes: {"/opt/myapp": "/opt/myapp_host"}
        expose: [80]
        ports: {8080: 80}
        environment: {"IS_MASTER": 1}

      c2:
        image: my_docker_image
        command: /bin/myapp
        volumes: ["/data"]
        links: {c1: master}

      c3:
        image: my_docker_image
        command: /bin/myapp
        links: {c1: master}
        memory: 100m

    network:
      flaky: 30%
      slow: 75ms 100ms distribution normal


The format is YAML and there are two important sections: ``containers`` and
``network``.

Containers
----------

Containers are described as a map with the key as the Blockade container name
(``c1``, ``c2``, ``c3`` in the example above). This key is used for commands
to manipulate the Blockade and is also used as the hostname of the container.

Each entry in the ``containers`` section is a single Docker container in the
Blockade. Each container parameter controls how the container is launched.
Most are simply pass-throughs to Docker. Many valuable details can be found
in the `Docker run`_ command documentation.

``image``
---------

``image`` is required and specifies the Docker image name to use for the
container. The image must exist in your Docker installation.

``command``
-----------

``command`` is optional and specifies the command to run within the container.
If not specified, a default command must be part of the image you are using.
You may include environment variables in this command, but to do so you must
typically wrap the command in a shell, like ``sh -c "/bin/myapp $MYENV"``.

``volumes``
-----------

``volumes`` is optional and specifies the volumes to mount in the container,
*from the host*. Volumes can be specified as either a map or a list. In map
form, the key is the path *on the host* to expose and the value is the
mountpoint *within the container*. In list form, the host path and container
mountpoint are assumed to be the same. See the `Docker volumes`_ documentation
for details about how this works.

``expose``
---------

``expose`` is optional and specifies ports to expose from the container. Ports
must be exposed in order to use the Docker `named links`_ feature.

``links``
---------

``links`` is optional and specifies links from one container to another. A
dependent container will be given environment variables with the parent
container's IP address and port information. See `named links`_ documentation
for details.

``ports``
---------

``ports`` is optional and specifies ports published to the host machine. It is
a dictionary from external port to internal container port.

``environment``
---------

``environment`` is optional and specifies environment variables for
``command``. See more details in ``command`` section above.

``hostname``
---------

``hostname`` is optional and gives the ability to redefine hostname of
a container.

``dns``
---------

``dns`` is optional and specifies a list of DNS-servers for container.

``start_delay``
---------------

``start_delay`` is optional and specifies a number of seconds to wait before
starting a container. This can be used as a stopgap way to ensure a dependent
service is running before starting a container.

``count``
---------
``count`` is optional and specifies the number of copies of the container to
launch.

``cap_add``
---------
``cap_add`` is optional and specifies additional root `capabilities`_


``memory``
---------
``memory`` is optional and specifies the maximum memory limit, mirrors the
``--memory`` `option <https://docs.docker.com/engine/admin/resource_constraints/#memory>`_
in ``docker run``.
If the limit is hit, the container will be shut down.


``container_name``
------------------

``container_name`` is optional and specifies a custom container name, instead
of letting blockade generate one. Use caution with this setting, because Docker
enforces uniqueness of names across all containers.

When this parameter is combined with ``count``, an underscore and index will
be suffixed to this name. For example "app" becomes "app_1", "app_2", etc.


Network
-------

The ``network`` configuration block controls the settings used for network
filter commands like ``slow`` and ``flaky``. If unspecified, defaults will
be used. There are two parameters:

``slow``
--------

``slow`` controls the amount and distribution of delay for network packets
when a container is in the Blockade slow state. It is specified
as an expression understood by the `tc netem`_ traffic control ``delay``
facility. See the man page for details, but the pattern is::

    TIME [ JITTER [ CORRELATION ] ]
        [ distribution { uniform | normal | pareto |  paretonormal } ]

``TIME`` and ``JITTER`` are expressed in milliseconds while ``CORRELATION``
is a percentage.

``flaky``
---------

``flaky`` controls the lossiness of network packets when a contrainer is in
the Blockade flaky state. It is specified as an expression understood by the
`tc netem`_ traffic control ``loss`` facility. See the man page for details,
but the simplified pattern is::

    random PERCENT [ CORRELATION ]

``PERCENT`` and ``CORRELATION`` are both expressed as percentages.

``driver``
----------

``driver`` specifies docker network stack. ``default`` will use standard Docker
networking, that allows to connect containers by links. Other option is ``udn``.
It will enable user defined network, that performs dns resolution of running
containers and allows to create any-to-any communications. In case of ``udn``
network environment variables with links will not bet set.

.. _Docker run: https://docs.docker.com/engine/reference/run/
.. _Docker volumes: https://docs.docker.com/engine/userguide/dockervolumes/
.. _named links: https://docs.docker.com/engine/userguide/networking/default_network/dockerlinks/
.. _tc netem: http://man7.org/linux/man-pages/man8/tc-netem.8.html
.. _capabilities: http://man7.org/linux/man-pages/man7/capabilities.7.html
