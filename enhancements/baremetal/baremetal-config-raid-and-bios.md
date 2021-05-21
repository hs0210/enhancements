---
title: Config-RAID-and-BIOS-for-Baremetal-IPI-deployments
authors:
  - "@hs0210"
reviewers:
  - TBD
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

# Config RAID and BIOS for Baremetal IPI deployments

## Release Signoff Checklist

- [ ] Enhancement is `implementable`
- [ ] Design details are appropriately documented from clear requirements
- [ ] Test plan is defined
- [ ] Operational readiness criteria is defined
- [ ] Graduation criteria for dev preview, tech preview, GA
- [ ] User-facing documentation is created in [openshift-docs](https://github.com/openshift/openshift-docs/)

## Summary

Currently, the installer-provisioned installation(IPI) deployments do not support the configuration of RAID and BIOS.
The proposal provides a feature to implement the configuration of RAID and BIOS during the IPI deployments for both
master and worker nodes.

## Motivation

As mentioned above, currently IPI deployments do not support the configuration of RAID and BIOS, if users want to
configure RAID and BIOS, they need to wait for the end of IPI deployments to re-create the node by modifying the **BMH**
to configure RAID and BIOS(The configuration of BIOS depends on the merge of [#302](https://github.com/metal3-io/baremetal-operator/pull/302)).
This brings too much trouble for users.

### Goals

- Allow users to configure RAID and BIOS in ***install-config.yaml***.
- IPI deployments can implement the configuration of RAID and BIOS.

### Non-Goals

- Vendor specific options regarding BIOS.

## Proposal

The configuration of RAID and BIOS is finally handed over to Ironic during the IPI deployments.
1. Add two new fields *raid* and *firmware* to *platform.baremetal.hosts* in the [install-config.yaml](https://github.com/openshift/installer/blob/master/data/data/install.openshift.io_installconfigs.yaml) file.
2. Openshift Installer process the field contents into manifests.
3. [Terraform-provider-ironic](https://github.com/openshift-metal3/terraform-provider-ironic) gets the
configuration of RAID and BIOS for master nodes and then passes it to Ironic([Baremetal Operator(BMO)](https://github.com/metal3-io/baremetal-operator)
has supported the process of RAID and BIOS for worker nodes).

### User Stories

With the addition of this feature, the users can configure RAID and BIOS in IPI deployments.

### Implementation Details/Notes/Constraints

#### Add two fields

New *platform.baremetal.hosts* fields called *raid* and *firmware*.

```yaml
platform:
  baremetal:
    ...
    hosts:
      - name: master-0
        role: master
        bmc:
          address: IPAddress
          username: UserName
          password: PassWord
        bootMACAddress: MACAddress
        raid: RAIDConfig
        firmware: BIOSConfig
```

RAID feature has been implemented in **BMO**, so the *raid* field here is the same as the [BMH]((https://github.com/metal3-io/baremetal-operator/blob/399f5ef7ee3831014c1425250bc4fa49641a8709/config/crd/bases/metal3.io_baremetalhosts.yaml)).
The *firmware* field is the same as the *spec.firmware* field in **BMH** which is been advancing by [#302](https://github.com/metal3-io/baremetal-operator/pull/302).

#### Process the fields in installer

For master nodes, call the `BuildTargetRAIDCfg` method in **BMO** to process the *raid* field into *target_raid_config*, and finally write *target_raid_config* into the ***terraform.baremetal.auto.tfvars.json*** file.

For worker nodes, copy *raid* and *firmware* field to **BMH**.

#### Process the fields in terraform-provider-ironic

Add *target_raid_config* and *bios_settings* fields to terraform-provider-ironic API, 
process the two fields by [manual cleaning](https://docs.openstack.org/ironic/latest/admin/cleaning.html#manual-cleaning).

### Risks and Mitigations

TBD

## Design Details

### Test Plan

- Unit tests for determining the configuration of RAID and BIOS passed to Ironic meeting expectations.
- e2e tests for determining the configuration of RAID and BIOS configured during the IPI deployments.

### Graduation Criteria

- Supporting the configuration of RAID and BIOS for worker nodes is target for GA in 4.9.
- Supporting the configuration of RAID and BIOS for master nodes is target for GA in 4.10.

### Upgrade / Downgrade Strategy

Older versions of the installer will ignore the *raid* and *firmware* fields.

## Implementation History

NONE

## Drawbacks

This will increase the number of steps in IPI deployments and take longer.

## Alternatives

The users can add RAID and BIOS configuration by modifying worker nodes' **BMH** called
**~/clusterconfigs/openshift/99_openshift-cluster-api_hosts- \*.yaml** generated after executing
`openshift-baremetal-install --dir ~/clusterconfigs create manifest`, so that the configuration
of RAID and BIOS for worker nodes can be done during the IPI deployments.

Using this approach needn't the modification of source code, but this approach is limited to worker nodes.
