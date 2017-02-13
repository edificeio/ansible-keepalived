ansible-keepalived
=========

Ansible role for installing and configuring keepalived (vrrp) on a cluster

Requirements
------------

Tested on Ubuntu 14.04 and Debian Jessie

Example usage
--------------


```
├── inventories
│   ├── group_vars
│   │   ├── lb-backup
│   │   └── lb-master
│   └── hosts
└── lb.yml
```

`inventories/hosts`:
```
[lb-master]
ent-core-2

[lb-backup]
ent-core-3

[lb:children]
lb-master
lb-backup
```

`inventories/group_vars/lb-master`:
```yaml
---
# lb-master
vrrp_script:
  script: "pidof nginx" # service check script, mandatory
  options: # optional
    - 'interval 2'
    - 'fall 2'
    - 'rise 2'

vrrp_instance:
  name: LB_77 # vrrp cluster name, mandatory
  interface: eth0 # vrrp cluster communication iface, mandatory
  state: MASTER # init
  priority: 200 # priority, set 50 above other members to be elected master. mandatory
  virtual_router_id: 77 # [1 - 255] vrrp id. mandatory
  unicast_src_ip: 192.168.56.78 # src ip address for vrrp communication, mandatory
  unicast_peer: ['192.168.56.79'] # ip address of vrrp cluster peers. mandatory
  virtual_ipaddress: ['10.0.0.1/16 dev eth0', '192.168.56.254/24 dev eth1'] # virtual ip addresses, mandatory
  virtual_routes: ['0.0.0.0/0 via 10.0.255.254 dev eth0'] # virtual routes, optional
  notify: '/opt/notify.sh' # script to be run on state change. the script is passer 3 parametrs (TYPE, INSTANCE, STATE). optional 
```

`inventories/group_vars/lb-backup`:
```yaml
---
# lb-backup
vrrp_script:
  script: "pidof nginx"
  options:
    - 'interval 2'
    - 'fall 2'
    - 'rise 2'

vrrp_instance:
  name: LB_77
  interface: eth0
  state: BACKUP
  priority: 100
  virtual_router_id: 77
  unicast_src_ip: 192.168.56.79
  unicast_peer: ['192.168.56.78']
  virtual_ipaddress: ['10.0.0.1/16 dev eth0', '192.168.56.254/24 dev eth1']
  virtual_routes: ['0.0.0.0/0 via 10.0.255.254 dev eth0']
  notify: '/opt/notify.sh'

```

`inventories/lb.yml`:
```yaml
---
- name: 'keepalived test'
  hosts: lb
  remote_user: root

  roles:
    - role: ansible-keepalived
```
And finally, run the playbook:
```bash
$ ansible-playbook -i inventories/hosts lb.yml
```


License
-------

MIT
