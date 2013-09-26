Docker Driver on Grizzly
========================

This repository contains files and instructions to get the [OpenStack Docker driver](https://wiki.openstack.org/wiki/Docker) working on Grizzly.

*Notes*:

 * This is not official OpenStack code. This is simply something I was tinkering with was able to successfully get working in my environment.
 * I'm using `nova-network` and `VlanManager` so the networking changes are specific toward that type of configuration.

Files
-----

* `driver.py` needs to replace the `nova/virt/docker/driver.py` file. A patch is included to more clearly see the changes that were made.
* `images.py` needs to replace the `glance/api/v1/images.py` file. A patch is also included.

Getting Docker to Work
----------------------

The above-linked wiki entry does a great job at explaining how this driver works. In addition, I found the following resources helpful:

* [Installing Docker](http://docs.docker.io/en/latest/installation/ubuntulinux/)
* [hypervisor-docker](https://github.com/openstack-dev/devstack/blob/master/lib/nova_plugins/hypervisor-docker): especially the [part](https://github.com/openstack-dev/devstack/blob/master/lib/nova_plugins/hypervisor-docker#L95-L100) where it shows how to launch the Docker registry and have it communicate with Glance.
* [install_docker.sh](https://github.com/openstack-dev/devstack/blob/master/tools/docker/install_docker.sh): supplemental installation info.
* [docker.sh](https://github.com/openstack-dev/devstack/blob/master/exercises/docker.sh): Launching a docker container in OpenStack.

Host Aggregates
---------------

By creating a Host Aggregate, you can designate a group of compute nodes to host Docker containers while having the rest of your compute nodes use your original hypervisor. [OpenStack Docs](http://docs.openstack.org/grizzly/openstack-compute/admin/content/host-aggregates.html), as usual, does an excellent job at explaining Host Aggregates and how to use them. For Docker:

Configure `nova-scheduler`:

```bash
$ grep scheduler_default_filters /etc/nova/nova.conf
scheduler_default_filters=AggregateInstanceExtraSpecsFilter,AvailabilityZoneFilter,RamFilter,ComputeFilter
```

Create a Host Aggregate:

```bash
$ nova aggregate-create docker nova
$ nova aggregate-set-metadata 1 docker=true
$ nova aggregate-add-host 1 node1.example.com
```

Now create a flavor and tie it to the aggregate:

```bash
$ nova flavor-create d1.tiny 500 1024 5 1
$ nova-manage instance_type set_key --name=d1.tiny --key=docker --value=true
```

Launching instances of type `d1.tiny` will cause OpenStack to launch the instance on `node1.example.com` only.
