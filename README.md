# ansible-qemu-kvm
Ansible role to provision virtual machines using the QEMU and KVM systems. 

Think of it like _OpenStack for poor people._

Currently, this role only creates virtual machines running the lite version of Ubuntu 18.04. Future versions may expand on this, but it is the primary server operating system that I use. 

## Usage

Add this role to the `roles` directory in your ansible project. 

Then, include the role using a top level playbook: 

```yaml
- name: KVM hosts 
  hosts: kvm-hosts
  become: true 
  roles: 
    - ansible-qemu-kvm
```



## Variables

This role requires these variables to exist in inventory: 

#### 1. Users

```yaml
users: 
- name: ongo
  full_name: Ongo Gablogian 
  passwd: $6$rounds=2048$aaaaaaaa
  pub_key: ssh-rsa AAAAB3N ongo@gablogianartcollection.org
```

This is a list of users that should be added to the server during provisioning. 

The `passwd` and `pub_key` parameters take your hashed password and your SSH public key so that you are able to access the server after provisioning. 

To generate a new password hash, use `mkpasswd -m sha-512 -R 2048` (Included in the whois package). 

#### 2. VMs

```yaml
virtual_machines: 
- name: u18-svr-001
  cpu: 1
  mem: 1024
  disk: 10G
  bridge: br10 
```

This is a list of virtual machines that should be built. 

Bridge: this is the name of the bridge device that references the vlan the server will be connected to. This must already exist on the KVM host. 

The virtual machine can also be built with a static IP address: 

```yaml
- name: u18-svr-002
  cpu: 1
  mem: 512 
  disk: 5G
  bridge: br10 
  net: 
    ip: 10.11.12.13/24
    gateway: 10.11.12.1 
    domain: gablogianartcollection.org
    dns: 
      - 1.1.1.1
      - 9.9.9.9
```

This variable takes exactly one IP address, domain, and gateway; and two or more DNS servers. 

#### Default Values

If no value is supplied, the default settings will be used: 

* CPU: 1 core
* Memory: 512 MB
* Disk: 5GB .qcow image
* Bridge: Default libvirt NAT network 
* Network: DHCP

## How it works

This role works by downloading the Ubuntu 18.04 cloud image, then converting it to Copy-On-Write format and cloning it for each virtual machine defined. 

When VMs are launched, they are given a small secondary disk image that includes a `cloud-config` file. This file is generated from a template that includes all the users and system parameters that are defined in inventory. There is also a network-config file that contains network metadata in Netplan-V2 format. 

Before running this, ensure that networking is correctly configured on the KVM hosts. 
