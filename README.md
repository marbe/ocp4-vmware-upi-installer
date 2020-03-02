# OpenShift 4 vmware UPI Installer
This repository contains [Ansible](https://www.ansible.com/) playbooks and tasks for [OpenShift](https://www.openshift.com/) 4 cluster installation on vmware with UPI mode.

The aim of this playbooks is to automate the UPI installation steps described into [Red Hat OpenShift documentation](https://docs.openshift.com/container-platform/4.3/installing/installing_vsphere/installing-vsphere.html)

## Pre-requisites
The minimun pre-requisite is a Red Hat / CentOS server to run this repo's playbooks on, called _bastion_ or _jump host_.

Ansible require version on _bastion host_:

* ansible > 2.8 is required
* These playbooks have been developed and tested with **ansible 2.9.3**.

DNS, DHCP and Load Balancers can either be installed with provided playbooks, or can be pre-provisioned as corporate infrastructure services. In this last case they must be configured ad described into [Red Hat OpenShift documentation](https://docs.openshift.com/container-platform/4.3/installing/installing_vsphere/installing-vsphere.html)

Some playbooks are provided in order to test that infrastructure pre-requisites are met, specifically those related to DNS records.

In order to provision VMs on vmware cluster, [pyvmomi](https://github.com/vmware/pyvmomi) python library is required on _bastion_. Its [installation](https://docs.ansible.com/ansible/latest/scenario_guides/vmware_scenarios/vmware_intro.html) is managed by an included  pre-requisites playbook.

### vSphere Permissions
vSphere permissions required from OpenShift installer to properly configure vsphere storageclass are detailed on [vmware vsphere storage for kubernetes page](https://vmware.github.io/vsphere-storage-for-kubernetes/documentation/vcp-roles.html).

vSphere permissions required by Ansible vmware_guest module are documented into [Notes section of vmware_guest page](https://docs.ansible.com/ansible/latest/modules/vmware_guest_module.html#notes).

Other useful information are provided within Ansible [VMware Guide](https://docs.ansible.com/ansible/latest/scenario_guides/guide_vmware.html).
## Provided Playbooks
Playbook name | Description
--- | ---
`ocp4-playbook-vmware-prereq.yaml` | Install vmware_gues prerequisites (currently pyvmomi).
`ocp4-playbook-cluster-create.yaml`| Setup the OpenShift 4 cluster manifests (ignition files, certificates) and creates VMs on vmware cluster. All VMs are created powered off.
`ocp4-playbook-poweron-vms.yaml`| Power on OpenShift VMs on vmware cluster.
`ocp4-playbook-erase-vms.yaml`| Power off and erase OpenShift  VMs on vmware cluster.
`ocp4-playbook-test-uri.yaml` | Test https get to URI required to install and use OpenShift.
`ocp4-playbook-test-dns.yaml`| Test for proper DNS records configuration.
`epel-enable-playbook.yaml`| Playbook to enable epel repository if needed (e.g. to install `ansible` from epel or `nagios-plugins-dhcp` to debug dhcp configuration).
`infrastructure-services-setup.yaml`| TO-DO: Playbook to install and configure infrastructure services (DNS, LB, DHCP) on _bastion host_ in order to completely automate UPI pre-requisites.

## How to use these playbooks to install your OpenShift 4 cluster
1. clone/download this repo on _bastion host_.
1. Provide an SSH keypair that will be configured to access CoreOS OCP nodes with `core` user.
1. customize `vars/ocp4-vars-vmware-upi-installer.yaml` var file with specific information related to your infrastructure.
1. test URI https get with playbook `ocp4-playbook-test-uri.yaml`
1. test DNS records configuration with playbook `ocp4-playbook-test-dns.yaml`
1. install ansible vmware_guest prerequisites with playbook `ocp4-playbook-vmware-prereq.yaml`
1. create OpenShift Cluster on vmware with playbook `ocp4-playbook-cluster-create.yaml`
1. poweron VMs with playbook `ocp4-playbook-poweron-vms.yaml`

All playbooks run on localhost. To run them, simply type:

`ansible-playbook <path_to_cloned_repo>/playbooks/<playbook-name>`

### OpenShift installation directory
To build openshift installation manifests and run openshift installer, this directory is created ad working directory:

`/tmp/openshift-install-<date +%Y%m%d>`

Also:
* `openshift-install`

and
* `oc`

binaries are installed under `/tmp/directory` in such a way that, after VMs poweron, you can follow and complete the installation with:

```
/tmp/openshift-install --dir=/tmp/openshift-install-<date +%Y%m%d> wait-for bootstrap-complete

export KUBECONFIG=/tmp/openshift-install-<date +%Y%m%d>/auth/kubeconfig

/tmp/oc patch configs.imageregistry.operator.openshift.io cluster --type merge --patch '{"spec":{"storage":{"emptyDir":{}}}}'

/tmp/oc patch configs.imageregistry.operator.openshift.io cluster --type merge --patch '{"spec":{"managementState": "Managed"}}'

/tmp/openshift-install --dir=/tmp/openshift-install-<date +%Y%m%d> wait-for install-complete
```

## How to contribute
Pull requests are wellcome!

Please provide your contributions by [branching](https://guides.github.com/introduction/flow/) master branch.

