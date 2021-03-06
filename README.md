﻿CloudFerry
==========

# Overview

CloudFerry is a tool for resources and workloads migration between OpenStack clouds. First of all CloudFerry tool
migrates cloud resources as tenants, users (preserving their passwords or generating new ones), roles, flavors and after
that, it transfers virtual workloads as instances with their own data (instance
image, root disk data, ephemeral drives, attached volumes) and network settings.


CloudFerry was tested on Openstack releases from Grizzly to Ice-House.
It supports migration process for clouds using any iSCSI-like mechanism for volumes or Ceph as a backend for
Cinder&Glance services, including case with Nova - Ephemeral volumes in Ceph.
Supported cases are listed below. Tool supports any iscsi-like mechanism for Cinder backend as for Cinder service with
LVM backend:


- 1) Source - Cinder (LVM), Glance (file) --> Destination - Cinder (LVM), Glance (file)
- 2) Source - Cinder & Glance (Ceph) --> Destination - Cinder (LVM), Glance (file)
- 3) Source - Cinder & Glance (Ceph) and Nova ephemeral volumes (Ceph) -->   Destination - Cinder (LVM), Glance (file)
- 4) Source - Cinder (LVM), Glance (file) --> Destination - Cinder & Glance (Ceph)
- 5) Source - Cinder (LVM), Glance (file) --> Destination - Cinder & Glance (Ceph) and Nova ephemeral volumes (Ceph)
- 6) Source - Cinder & Glance (Ceph) --> Destination - Cinder & Glance (Ceph)
- 7) Source - Cinder & Glance (Ceph) and Nova ephemeral volumes (Ceph) -->   Destination - Cinder & Glance (Ceph) and
Nova ephemeral volumes (Ceph)


Also CloudFerry can migrate instances, which were booted from bootable  volumes with the same storage backends as in
previous listed cases.


CloudFerry uses External network as a transfer network, so you need to have a connectivity from host where you want
to execute the tool (transition zone) to both clouds through external network.
CloudFerry can migrate instances from clouds with nova-network or quantum/neutron to new cloud with neutron network
service. Also the tool can transfer instances in to the fixed networks with the same CIDR (they will be found
automatically) or list new networks for instances in config.yaml file in overwrite section.


Cloudferry also allow keep ip addresses of instances and transfer security groups (with rules) with automatic detection
of network manager on source and destination clouds (quantum/neutron or nova).


All functions are configured in yaml file, you can see the examples in configs directory.
At the moment config file does not support default values so you need to set up all settings manually. Please note,
that if any valuable setting appeared to be missing in config file, process will crash. Default settings is planned
to implement in nearest future.


# Requirements

 - Connection to source and destination clouds through external(public) network from host with CloudFerry.
 - Valid private ssh-key for both clouds which will be using by CloudFerry for data transferring.
 - Admin keystone access (typically admin access point lives on 35357 port).
 - sudo/root access on compute and controller nodes.
 - Openstack MySQL DB write access.
 - Credentials of global cloud admin for both clouds.
 - All the Python requirements are listed in requirements.txt.


# Installation
Cloudferry can be prepared and installed as docker container.

## Building the docker container
```
docker build --build-arg cf_commit_or_branch=origin/master -t <username>/cf-in-docker .
```

## Container running
```
docker run -it <username>/cf-in-docker
```

## Saving and loading the container files
```
docker save --output=/path/to/save/CloudFerry.img <username>/cf-in-docker
docker load --input=/path/to/save/CloudFerry.img
```

# Usage

## Overview
CloudFerry tool is used by running python fabric scripts from the CloudFerry directory:
```
# see list of available commands
cloudferry --help
```

## Configuration

You can generate sample configs by:
    ```
    mkdir my_migration
    cd my_migration
    cloudferry init
    ```
Configuration process is quite complex and mostly manual try-and-see-if-works process. Configuration documentation
is TBD.

## Whole cloud migration
Use `migrate` command with config file specified:

```
cloudferry migrate <config file>
```

## Migrating specific instances

In order to migrate specific VMs, one should use filters. This is done through modifying filters file
(`configs/filter.yaml` by default).

Edit `configs/filter.yaml`:
```
instances:
    id:
        - 7c53a6ab-0149-4232-80b3-b2d7ce02995a
        - f0fea76a-0a7d-4c25-ab9e-f048dbc7365d
```

Run migration as usual:
```
cloudferry migrate <config file>
```

## Playground

See QUICKSTART.md for the quickest way of running your first successful migration.
