# ansible-role-proxmox-firewall

This role is used to manage the Proxmox VE firewall.

Because the firewall configuration is very different from site to site this is only an example. You need to know how you want to configure your network.

Proxmox uses one file called `cluster.fw` to define all aliases, IP-sets groups and rules. This file also includes the applied rules for the hosts and the cluster itself. 
Then there a per VM one firewall configuration like `100.fw`  where you define per VM which rule is assigned to KVM ID 100.

I asume the following inventory. There is a group of Proxmox hosts inside `proxmox`. Because I use GlusterFS, there is a group `gluster` too (which contains the same server here, but it could be different). 
Then there is group of `kvm` nodes. All kvms have a `vmid`. This value could be read via proxmox-kvm module in Ansible (this is not yet supported). So I identify the KVM id with that value hardcoded.
Also there is a dictionay containing the IPv4 and IPv6 settings of all known nodes. This is necessary, because during bootstrap it is very possible that you don't have access to this or it changes during runtime (like installing docker0 changes ansible_default_ipv4). Often you know this values before running Ansible the first time.

    all:
      hosts:
        proxmox1:
          ansible_host: 10.0.0.1
          network:
            ipv4:
              address: 10.0.0.1
            ipv6:
              address: fd01:0:0:1::
              prefix: 64
        proxmox2:
          ansible_host: 10.0.0.2
          network:
            ipv4:
              address: 10.0.0.2
            ipv6:
              address: fd01:0:0:2::
              prefix: 64
        kvm1:
          ansible_host: 10.0.1.1
          vmid: 100
          network:
            ipv4:
              address: 10.0.1.1
            ipv6:
              address: fd01:0:1:1::
              prefix: 64
        kvm2:
          ansible_host: 10.0.1.2
          vmid: 101
          network:
            ipv4:
              address: 10.0.1.2
            ipv6:
              address: fd01:0:1:2::
              prefix: 64
        kvm3:
          ansible_host: 10.0.1.1
          vmid: 102
          network:
            ipv4:
              address: 10.0.1.3
            ipv6:
              address: fd01:0:1:3::
              prefix: 64
        kvm4:
          ansible_host: 10.0.1.4
          vmid: 103
          network:
            ipv4:
              address: 10.0.1.4
            ipv6:
              address: fd01:0:1:4::
              prefix: 64
        kvm5:
          ansible_host: 10.0.1.5
          vmid: 104
          network:
            ipv4:
              address: 10.0.1.5
            ipv6:
              address: fd01:0:1:5::
              prefix: 64
        vpn1:
          ansible_host: 10.0.2.1
          network:
            ipv4:
              address: 10.0.2.1
            ipv6:
              address: fd01:0:2:1::
              prefix: 64
      children:
        proxmox:
          hosts:
            proxmox1:
            proxmox2:
        gluster:
          hosts:
            proxmox1:
            proxmox2:
        vms:
          hosts:
            kvm1:
            kvm2:
        db:
          hosts:
            kvm3:
        mail:
          hosts:
            kvm4:
        monitoring:
          hosts:
            kvm4:
        loadbalancer:
          hosts:
            kvm5:
        vpn:
          hosts:
            vpn1:


Then this a an example playbook:

    - name: "Install Proxmox"
      hosts: proxmox
      roles:
        - trw.proxmox-firewall
