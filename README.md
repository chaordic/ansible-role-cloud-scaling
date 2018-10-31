cloud-scaling
=============

Ansible role to manage applications' resources on Cloud environments.

Requirements
------------

> TODO

Role Variables
--------------

`cs_version`: Global version to create resources, AMI and LC.
`cs_global_tags`: Global tags to apply on the resources
`cs_lc_default`: Global options for LC module
`cs_asg_default`: Global options for ASG
`cloud_scaling_groups`: a list of scaling groups

Dependencies
------------

Ansible 2.5+

Example Playbook
----------------


    - hosts: servers
      roles:
         - { role: username.rolename, x: 42 }

License
-------

GPLv3

Author Information
------------------
