To setup ethernet interfaces, vlan interfaces, bridge interfaces on the NFV server
```
$ ansible-playbook -i ~/iac-iith-ccs-ws006/inventory/hosts.yml -e "target_host=ccs-ws006" playbooks/setup_ethernets.yml
```

```
$ ansible-playbook -i ~/iac-iith-ccs-ws006/inventory/hosts.yml -e "target_host=ccs-ws006" playbooks/setup_bridges.yml
```

To download qcow2 image to the host (bare-metal server or workstation)
```
# cd /var/lib/libvirt/images/
$ wget https://cdimage.debian.org/images/cloud/bookworm/20260210-2384/debian-12-generic-amd64-20260210-2384.qcow2
```

To setup NAT and VPN router VMs
```
$ ansible-playbook -i ~/iac-iith-ccs-ws006/inventory/hosts.yml -e 'target_host=ccs-ws006' -e '{"vms_to_provision": ["natrtr", "vpnrtr"]}' playbooks/setup_vms.yml
```

```
$ ansible-playbook -i ~/iac-iith-ccs-ws006/inventory/hosts.yml -e 'target_host="natrtr"' -e 'ansible_ssh_common_args="-o ProxyJump=maruthisi@198.18.1.106"' playbooks/setup_nat_router_using_linux_kernel.yml
```

```
$ ansible-playbook -i ~/iac-iith-ccs-ws006/inventory/hosts.yml -e 'target_host="vpnrtr"' -e 'ansible_ssh_common_args="-o ProxyJump=maruthisi@198.18.1.106"' playbooks/setup_wg_vpn_router_using_linux_kernel.yml
```

To setup Core router and DNS server LXDs
```
$ ansible-playbook -i ~/iac-iith-ccs-ws006/inventory/hosts.yml -e 'target_host=ccs-ws006' -e '{"lxds_to_provision": ["corertr", "dnssrv", "ldapsrv"]}' playbooks/setup_lxds.yml
```

```
$ ansible-playbook -i ~/iac-iith-ccs-ws006/inventory/hosts.yml -e 'target_host="dnssrv"' -e 'ansible_ssh_common_args="-o ProxyJump=maruthisi@198.18.1.106"' playbooks/setup_dns_server_bind9.yml
```


```
$ ansible-playbook -i ~/iac-iith-ccs-ws006/inventory/hosts.yml -e 'target_host="corertr"' -e 'ansible_ssh_common_args="-o ProxyJump=maruthisi@198.18.1.106"' playbooks/setup_core_router_using_linux_kernel.yml
```

Setting up LDAP server (OpenLDAP) on a given target {#ldap-server}

To generate private key for CA certificate on ansible control host

```
$ sudo apt install gnutls-bin
$ certtool --generate-privkey --bits 4096 --outfile ./ldapsrv-ca-key.pem
$ vi ./ldapsrv-ca-key.pem
-----BEGIN RSA PRIVATE KEY-----
<copy the private key>
-----END RSA PRIVATE KEY-------

$ ansible-vault edit vault/infra_ca_private_key.yml
infra_ca_private_key: |
  -----BEGIN RSA PRIVATE KEY-----
  <paste the private key>
  -----END RSA PRIVATE KEY-------
```

Now setup LDAP server in the given target

```
$ ansible-playbook -i ~/iac-iith-ccs-ws006/inventory/hosts.yml -e '{"target_host":"ldapsrv"}' -e 'ansible_ssh_common_args="-o ProxyJump=maruthisi@198.18.1.106"' --ask-vault-pass playbooks/setup_ldap_server_openldap.yml
Vault password:
```

To setup LDAP groups on the LDAP server
```
$ ansible-playbook -i ~/iac-iith-ccs-ws006/inventory/hosts.yml  -e 'ansible_ssh_common_args="-o ProxyJump=maruthisi@198.18.1.106"' -e 'target_host=ldapsrv' --ask-vault-pass playbooks/setup_ldap_server_groups.yml
Vault password:
```

To setup LDAP users on the LDAP server
```
$ ansible-playbook -i ~/iac-iith-ccs-ws006/inventory/hosts.yml -e 'ansible_ssh_common_args="-o ProxyJump=maruthisi@198.18.1.106"' -e 'target_host=ldapsrv' --ask-vault-pass playbooks/setup_ldap_server_users.yml
Vault password:
```

To setup LDAP client on targets
```
$ ansible-playbook -i ~/iac-iith-ccs-ws006/inventory/hosts.yml -e 'ansible_ssh_common_args="-o ProxyJump=maruthisi@198.18.1.106"' -e 'target_host=ldap_clients' playbooks/setup_ldap_client.yml
```


To setup IPAM server LXD
```
$ ansible-playbook -i ~/iac-iith-ccs-ws006/inventory/hosts.yml -e 'target_host=ccs-ws006' -e '{"lxds_to_provision": ["ipamsrv"]}' playbooks/setup_lxds.yml
```

```
$ openssl rand -base64 50
7TAgFVjThwqPzKNCAUEYp6PpHjbmbwTki74hIFez/p1W6wa9JjoEJlV2pjGceZNG
7bk=

$ ansible-vault create ~/iac-iith-ccs-ws006/inventory/ipamsrv/netbox.vault.yml
netbox_secret_key: "7TAgFVjThwqPzKNCAUEYp6PpHjbmbwTki74hIFez/p1W6wa9JjoEJlV2pjGceZNG7bk="
netbox_db_password: "netbox_password_change_me"

$ ansible-playbook -i ~/iac-iith-ccs-ws006/inventory/hosts.yml -e 'target_host="ipamsrv"' --ask-vault-pass playbooks/setup_netbox_server.yml
```

To setup API server LXD
```
$ ansible-playbook -i ~/iac-iith-ccs-ws006/inventory/hosts.yml -e 'target_host=ccs-ws006' -e '{"lxds_to_provision": ["apisrv"]}' playbooks/setup_lxds.yml
```

To setup hosts in LAN
```
$ ansible-playbook -i ~/iac-iith-ccs-ws006/inventory/hosts.yml -e 'target_host=ccs-ws006' -e '{"lxds_to_provision": ["lanhost0", "lanhost1"]}' playbooks/setup_lxds.yml
```


