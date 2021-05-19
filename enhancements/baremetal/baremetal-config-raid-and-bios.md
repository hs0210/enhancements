---
title: Config-raid-and-bios-for-Baremetal-IPI-deployments
authors:
  - "@hs0210"
  - "@"
reviewers:
  - TBD
  - "@alicedoe"
approvers:
  - TBD
creation-date: 2021-05-18
last-updated: yyyy-mm-dd
status: provisional
see-also:
  - 
replaces:
  -
superseded-by:
  -　https://github.com/metal3-io/baremetal-operator/pull/302
---

# Config raid and bios for Baremetal IPI deployments

## Release Signoff Checklist

- [ ] Enhancement is `implementable`
- [ ] Design details are appropriately documented from clear requirements
- [ ] Test plan is defined
- [ ] Operational readiness criteria is defined
- [ ] Graduation criteria for dev preview, tech preview, GA
- [ ] User-facing documentation is created in [openshift-docs](https://github.com/openshift/openshift-docs/)

## Summary

This proposal is dedicated to making installer-provisioned installation(IPI) deployments
support the configuration of RAID and BIOS for both master and worker nodes.

Currently, IPI does not support the configuration of RAID and BIOS. The proposal provides a feature to complete the
configuration of raid and bios during the IPI deployments.

## Motivation

As mentioned above, currently IPI deployments do not support the configuration of RAID and BIOS, if users want to
config RAID and BIOS, they need to wait for the end of IPI deployments to re-create the node by modifying the BMH
file to achieve the configuration of RAID and BIOS(The configuration of BIOS depends on the merge of [#302](https://github.com/metal3-io/baremetal-operator/pull/302)).
There is no doubt that this is too much trouble for users.

### Goals

- Allow users to config RAID and BIOS in `install-config.yaml`.
- IPI deployments can implement the configuration of RAID and BIOS.

### Non-Goals

- Vendor specific options regarding BIOS.

## Proposal

Two new fields `RAID` and `BIOS` will be added to `platform.baremetal.hosts` in the [install-config.yaml](https://github.com/openshift/installer/blob/master/data/data/install.openshift.io_installconfigs.yaml) file.

### User Stories

With the addition of this feature, the users can achieve the configuration of RAID and BIOS in IPI deployments.

### Implementation Details/Notes/Constraints [optional]

The value of `RAID` and `BIOS` fields will be parsed to manifests,

### Risks and Mitigations

TBD

## Design Details

### Test Plan

- Unit tests for determining the configuration of RAID and BIOS to pass to ironic.
- e2e tests for determining the configuration of RAID and BIOS configured in the IPI deployments.

### Graduation Criteria

- Supporting the configuration of RAID and BIOS for worker nodes is target for GA in 4.9.
- Supporting the configuration of RAID and BIOS for master nodes is target for GA in 4.10.

### Upgrade / Downgrade Strategy

Before version 4.9, if RAID and BIOS fields are added to the `install-config.yaml` file,
an error will be reported when creating the OpenShift Container Platform manifests.
After version 4.9, if there is no RAID and BIOS fields in the yaml file, the process
of IPI is the same as the previous version.

## Implementation History

NONE

## Drawbacks

This will increase the number of steps in IPI deployment and take longer.

## Alternatives

The users can add RAID and BIOS configuration by modifying worker nodes' BMH definition file called
`~/clusterconfigs/openshift/99_openshift-cluster-api_hosts- *.yaml` generated after executing
`openshift-baremetal-install --dir ~/clusterconfigs create manifest`, so that the configuration
of RAID and BIOS for worker nodes can be done in the IPI process.

There is no code level change using this approach, but this approach is limited to worker nodes, not to master nodes.
Because the BMH file of the worker node is processed by [Baremetal Operator(BMO)](https://github.com/metal3-io/baremetal-operator), the BMH file of the master node
is processed by [terraform-provider-ironic](https://github.com/openshift-metal3/terraform-provider-ironic) which currently does not support raid and bios processing.
