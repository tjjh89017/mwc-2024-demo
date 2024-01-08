# VyOS MWC 2024

## reqirement

- GNS3>=2.2
- Cloud-init with NoCloud QCOW2
    - `ansible-playbook qemu.yml -e iso_local=/vbuild/vyos_image.iso -e disk_size=4 -e cloud_init=true -e cloud_init_ds=NoCloud,ConfigDrive,None -e guest_agent=qemu -e grub_console=serial`
- config.raw with simple config
- Check deployment debian's `isc-dhcp-server` config and GNS3 VM's MAC address

### Config.raw files

- meta-data (empty)
- network-config (disable all dhcp by default)
```yaml
version: 2
ethernets:
  eth0:
    dhcp4: false
    dhcp6: false
```
- user-data
    - set password "vyos"
    - set vrf and dhcp for the first port as OOB mgmt
```
#cloud-config
vyos_config_commands:
  - set system login user vyos authentication encrypted-password '$6$rounds=656000$pV10YW6kOFKS3T8S$OyvGsNjWa7Rc/uuW3f8sVDrCvtOe3PVUlq31A7lH4vPuSBA51yKAUpm0HfjLiUfZQggdzSBt3AgO1zB/aKTnA/'
  - set interfaces ethernet eth0 address 'dhcp'
  - set interfaces ethernet eth0 vrf 'mgmt'
  - set vrf bind-to-all
  - set vrf name mgmt table '100'
```

## Usage

- login deployment debian (debian / debian)
- cd mwc-2024-demo
- clean up all devices
    - `ansible-playbook cleanup.yml`
    - and wait for reboot
- demo VPN and SD-wan setup
    - `ansible-playbooy demo-vpn.yml`
    - wait for all success
    - disconnect some cables, and test PC connection to each other
    - login into some vyos, show CLI and destroy some config, and use Ansible cleanup and demo-vpn to rebuild the demo
        - `ansible-playbook -l 172.31.0.13 cleanup.yml demo-vpn.yml`

## Demo VPN

- BGP
- NAT
- failover
- BGP over Wireguard VPN
- Ansible auto deployment
- Cloud-init init script demo
- CLI demo
- fast replacement demo

### Architecture

- All devices are using OOB as mgmt port for ansible deployment
- ISP1 and ISP2 are internet emulation
- ISP1 and ISP2 have BGP sessions with each other
- HOME, HQ, BRANCH are all NATed to ISP1 or ISP2
- HOME, HQ, BRANCH are office router with two failover uplink
- HOME, HQ, BRANCH are wireguard connected (full mesh)
- HOME, HQ, BRANCH's BGPs are over wireguard tunnels (failover)

### NOTE

- boot deployment first and wait for dhcp server ready
    - Otherwise, VyOS might not get DHCP IP immediately
