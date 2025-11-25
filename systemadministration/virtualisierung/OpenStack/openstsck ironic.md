---
tags:
  - openstack
  - ironic
---
https://docs.openstack.org/ironic/5.1.3/deploy/install-guide.html
[ironic](https://docs.openstack.org/ironic/latest/install/get_started.html)
https://opendev.org/openstack/ironic/src/commit/5592b1fa9a9cc76a77aa695492e089adcba51573/doc/source/deploy/install-guide.rst
https://github.com/openstack/bifrost

# ironic baremetal

https://ironicbaremetal.org/


https://docs.redhat.com/en/documentation/red_hat_openstack_platform/9/html/bare_metal_provisioning/install_and_configure_openstack_bare_metal_provisioning_ironic#install_and_configure_openstack_bare_metal_provisioning_ironic

# Bifrost Installation

Bifrost (pronounced bye-frost) is a set of Ansible playbooks that automates the task of deploying a base image onto a set of known hardware using [ironic](https://docs.openstack.org/ironic/latest/). It provides modular utility for one-off operating system deployment with as few operational requirements as reasonably possible.

Use cases include:

- Installation of ironic in standalone/noauth mode without other OpenStack components.
- Deployment of an operating system to a known pool of hardware as a batch operation.
- Testing and development of ironic in the standalone mode.

https://docs.openstack.org/bifrost/2025.1/install/index.html
https://docs.openstack.org/bifrost/latest/install/index.html