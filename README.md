## Hiera-based OpenVox ENC
======================

This project acts as a miniature OpenVox ENC in YAML files, made
accessible by Hiera. This makes server-side node configuration
(e.g. environments, custom node parameters) easy to edit in the same
way (and same place) as normal Hiera-based OpenVox data.

Since an exec-based ENC simply prints out YAML when passed a node name,
the script just gets the merged hierarchical data with Hiera and then
prints the resulting output.

The project is intended to be used in an envrionment based that is subdivdied
by roles, but provides the flexibility to impelment it as a flater implementation
It is a fork of [puppet-hiera-enc](https://github.com/Zetten/puppet-hiera-enc) 

### Usage
-----

Deploy this repository on the openvox server as `/etc/openvox/hiera-enc`,
and add the following lines to `puppet.conf`:

    [master]
        node_terminus = exec
        external_nodes = /etc/openvox/hiera-enc/enc

The node hierarchy is controlled by `/etc/puppet/hiera-enc/host-hiera.yaml`
The `domain` and `clientcert` variables are passed from the CLI invocation
to this Hierarchy

The role hierarchy is controlled by `/etc/puppet/hiera-enc/role-hiera.yaml`
The `domain`, `clientcert` and `role` variables are passed to this hierarchy. 
The `role` values are from the `roles` array. 

#### How are roles defined, and used 
-----

Roles are defined as a hash. They can be defined on any data source
within the host or role hiera and are recursively loaded. You can negate the 
loading of a role. If you negate the loading of a role it superceeds all
configured heirarchy. See below for the expected format of the role definitions

```
---
roles:
 login: true
 cvmfs: false
 slurm: false
 dtn: true
```

Roles are intended to be a combination of classes and parameters that contain, or
define a complete service, but can be utlized (or not) however you would like. 

Check the data returned for a given node by executing `./enc <fqdn>`,
just as OpenVox itself will do. Note that the `DEBUG` lines are printed 
to STDERR and are not parsed by OpenVox.

Example
-------

Assume the yaml files `hostname.example.com.yaml` and `roles/linux.yaml`:

    ---
    roles:
      linux: true
      ssh: false
    parameters:
      type: workstation
<!-- -->
    --- ## roles/linux.yaml ##
    environment: production
    roles:
      ssh: true

In the above example the ssh role would be excluded from the final results because it has a "false" 
even though the included linux role has `ssh: true`. 

When the enc is queried for `hostname.example.com`, the result is the
merged hash of these values:

    $ ./enc hostname.example.com
    DEBUG: 2014-07-11 13:04:24 +0200: Hiera YAML backend starting
    DEBUG: 2014-07-11 13:04:24 +0200: Looking up parameters in YAML backend
    DEBUG: 2014-07-11 13:04:24 +0200: Looking for data source hostname.example.com
    DEBUG: 2014-07-11 13:04:24 +0200: Found parameters in hostname.example.com
    DEBUG: 2014-07-11 13:04:24 +0200: Looking up environment in YAML backend
    DEBUG: 2014-07-11 13:04:24 +0200: Looking for data source hostname.example.com
    DEBUG: 2014-07-11 13:04:24 +0200: Looking for data source default
    DEBUG: 2014-07-11 13:04:24 +0200: Found environment in default
    ---
    environment: R20250701
    parameters:
     servicegroup: Management
     location: CloudWest
     enc_roles:
      - Base
      - Management
    classes:
     - base
     - security::access
     - accounts
     - firewall
     - puppet
