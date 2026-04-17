To setup ethernet interfaces, vlan interfaces, bridge interfaces on the NFV server
<pre>
```
$ ansible-playbook -i ~/iac-iith-ccs-ws006/inventory/hosts.yml -e "target_host=ccs-ws006" playbooks/setup_ethernets.yml
```
</pre>

<pre>
```
$ ansible-playbook -i ~/iac-iith-ccs-ws006/inventory/hosts.yml -e "target_host=ccs-ws006" playbooks/setup_bridges.yml
```
</pre>

To download qcow2 image to the host (bare-metal server or workstation)
<pre>
```
# cd /var/lib/libvirt/images/
$ wget https://cdimage.debian.org/images/cloud/bookworm/20260210-2384/debian-12-generic-amd64-20260210-2384.qcow2
```
</pre>

To setup NAT and VPN router VMs
<pre>
```
$ ansible-playbook -i ~/iac-iith-ccs-ws006/inventory/hosts.yml -e 'target_host=ccs-ws006' -e '{"vms_to_provision": ["natrtr", "vpnrtr"]}' playbooks/setup_vms.yml
```
</pre>

<pre>
```
$ ansible-playbook -i ~/iac-iith-ccs-ws006/inventory/hosts.yml -e 'target_host="natrtr"' -e 'ansible_ssh_common_args="-o ProxyJump=maruthisi@198.18.1.106"' playbooks/setup_nat_router_using_linux_kernel.yml
```
</pre>

<pre>
```
$ ansible-playbook -i ~/iac-iith-ccs-ws006/inventory/hosts.yml -e 'target_host="vpnrtr"' -e 'ansible_ssh_common_args="-o ProxyJump=maruthisi@198.18.1.106"' playbooks/setup_wg_vpn_router_using_linux_kernel.yml
```
</pre>

To setup Core router and DNS server LXDs
<pre>
```
$ ansible-playbook -i ~/iac-iith-ccs-ws006/inventory/hosts.yml -e 'target_host=ccs-ws006' -e '{"lxds_to_provision": ["corertr", "dnssrv", "ldapsrv"]}' playbooks/setup_lxds.yml
```
</pre>

<pre>
```
$ ansible-playbook -i ~/iac-iith-ccs-ws006/inventory/hosts.yml -e 'target_host="dnssrv"' -e 'ansible_ssh_common_args="-o ProxyJump=maruthisi@198.18.1.106"' playbooks/setup_dns_server_bind9.yml
```
</pre>


<pre>
```
$ ansible-playbook -i ~/iac-iith-ccs-ws006/inventory/hosts.yml -e 'target_host="corertr"' -e 'ansible_ssh_common_args="-o ProxyJump=maruthisi@198.18.1.106"' playbooks/setup_core_router_using_linux_kernel.yml
```
</pre>

Setting up LDAP server (OpenLDAP) on a given target {#ldap-server}

To generate private key for CA certificate on ansible control host

<pre>
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
</pre>

Now setup LDAP server in the given target

<pre>
```
$ ansible-playbook -i ~/iac-iith-ccs-ws006/inventory/hosts.yml -e '{"target_host":"ldapsrv"}' -e 'ansible_ssh_common_args="-o ProxyJump=maruthisi@198.18.1.106"' --ask-vault-pass playbooks/setup_ldap_server_openldap.yml
Vault password:
```
</pre>

To setup LDAP groups on the LDAP server
<pre>
```
$ ansible-playbook -i ~/iac-iith-ccs-ws006/inventory/hosts.yml  -e 'ansible_ssh_common_args="-o ProxyJump=maruthisi@198.18.1.106"' -e 'target_host=ldapsrv' --ask-vault-pass playbooks/setup_ldap_server_groups.yml
Vault password:
```
</pre>

To setup LDAP users on the LDAP server
<pre>
```
$ ansible-playbook -i ~/iac-iith-ccs-ws006/inventory/hosts.yml -e 'ansible_ssh_common_args="-o ProxyJump=maruthisi@198.18.1.106"' -e 'target_host=ldapsrv' --ask-vault-pass playbooks/setup_ldap_server_users.yml
Vault password:
```
</pre>

To setup LDAP client on targets
<pre>
```
$ ansible-playbook -i ~/iac-iith-ccs-ws006/inventory/hosts.yml -e 'ansible_ssh_common_args="-o ProxyJump=maruthisi@198.18.1.106"' -e 'target_host=ldap_clients' playbooks/setup_ldap_client.yml
```
</pre>


To setup IPAM server LXD
<pre>
```
$ ansible-playbook -i ~/iac-iith-ccs-ws006/inventory/hosts.yml -e 'target_host=ccs-ws006' -e '{"lxds_to_provision": ["ipamsrv"]}' playbooks/setup_lxds.yml
```
</pre>

<pre>
```
$ openssl rand -base64 50
7TAgFVjThwqPzKNCAUEYp6PpHjbmbwTki74hIFez/p1W6wa9JjoEJlV2pjGceZNG
7bk=

$ ansible-vault create ~/iac-iith-ccs-ws006/inventory/ipamsrv/netbox.vault.yml
netbox_secret_key: "7TAgFVjThwqPzKNCAUEYp6PpHjbmbwTki74hIFez/p1W6wa9JjoEJlV2pjGceZNG7bk="
netbox_db_password: "netbox_password_change_me"

$ ansible-playbook -i ~/iac-iith-ccs-ws006/inventory/hosts.yml -e 'target_host="ipamsrv"' --ask-vault-pass playbooks/setup_netbox_server.yml
```
</pre>

To setup API server LXD
<pre>
```
$ ansible-playbook -i ~/iac-iith-ccs-ws006/inventory/hosts.yml -e 'target_host=ccs-ws006' -e '{"lxds_to_provision": ["apisrv"]}' playbooks/setup_lxds.yml
```
</pre>

To setup hosts in LAN
<pre>
```
$ ansible-playbook -i ~/iac-iith-ccs-ws006/inventory/hosts.yml -e 'target_host=ccs-ws006' -e '{"lxds_to_provision": ["lanhost0", "lanhost1"]}' playbooks/setup_lxds.yml
```
</pre>


