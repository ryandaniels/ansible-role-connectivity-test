# ansible-role-connectivity-test

Use nc (netcat) to test for network issues to specified IP and port (TCP or UDP).  
Dynamically determines source IP to use based on destination IP from available route.

Default uses TCP protocol. You can specify `protocol: udp` if UDP is needed for test.

## Distros tested

* RHEL/CentOS 7.x
* RHEL/CentOS 5.9

## Dependencies

OS commands:  

* nc (netcat) - will be installed if not present
* ip
* awk

## Default Settings

* Enable debug

```yaml
debug_enabled_default: false
```

* Timeout

```yaml
connectivity_test_wait_default: 5
```

* UDP argument to use if 'udp' specified in config

```yaml
connectivity_test_udp_command: "-u"
```

* Proxy

```yaml
proxy_env: []
```

## Example config file (group_vars/xxx/connectivity-test-vars.yml)

```yaml
---
connectivity_test_destinations:
  - { ip: 192.168.56.10, port: 22 }
  - { ip: 192.168.56.10, port: 5000 }
  - { ip: 192.168.56.13, port: 443 }
  - { ip: 192.168.56.13, port: 67, protocol: udp }
```

## Example Playbook connectivity-test.yml

```yaml
---
- hosts: '{{inventory}}'
  become: yes
  roles:
  - connectivity-test
```

## Usage

```bash
ansible-playbook connectivity-test.yml --extra-vars "inventory=all-dev" -i hosts
```

Skip installing packages (if known already there - speeds up task)

```bash
ansible-playbook connectivity-test.yml --extra-vars "inventory=all-dev" -i hosts --skip-tags connectivity_install_pkg
```

## Example output

```ansible
TASK [setup] *******************************************************************
ok: [centos5]
ok: [centos7]

TASK [connectivity-test : Install dependant packages in RedHat based machines] *
ok: [centos5] => (item=[u'nc'])
ok: [centos7] => (item=[u'nc'])

TASK [connectivity-test : Get source IP based on destination IP] ***************
ok: [centos7] => (item=source ip: 192.168.56.13, destination ip: 192.168.56.10)
ok: [centos5] => (item=source ip: 192.168.56.10, destination ip: 192.168.56.10)
ok: [centos7] => (item=source ip: 192.168.56.13, destination ip: 192.168.56.10)
ok: [centos5] => (item=source ip: 192.168.56.10, destination ip: 192.168.56.10)
ok: [centos7] => (item=source ip: 192.168.56.13, destination ip: 192.168.56.13)
ok: [centos5] => (item=source ip: 192.168.56.10, destination ip: 192.168.56.13)

TASK [connectivity-test : debug] ***********************************************

TASK [connectivity-test : test network connectivity] ***************************
ok: [centos7] => (item=source ip: 192.168.56.13, destination ip: 192.168.56.10, port: 22, protocol: tcp, time: 2017-02-22 13:21:23.044606, stdout: SSH-2.0-OpenSSH_4.3, stderr: Ncat: Version 6.40 ( http://nmap.org/ncat )
Ncat: Connected to 192.168.56.10:22.
Ncat: 0 bytes sent, 20 bytes received in 0.01 seconds.

)
ok: [centos5] => (item=source ip: 192.168.56.10, destination ip: 192.168.56.10, port: 22, protocol: tcp, time: 2017-02-22 13:21:22.498293, stdout: SSH-2.0-OpenSSH_6.5
Connection to 192.168.56.10 22 port [tcp/ssh] succeeded!, stderr:

)
failed: [centos7] => (item=source ip: 192.168.56.13, destination ip: 192.168.56.10, port: 5000, protocol: tcp, time: 2017-02-22 13:21:23.307189, stdout: , stderr: Ncat: Version 6.40 ( http://nmap.org/ncat )
Ncat: Connection refused.

)
failed: [centos5] => (item=source ip: 192.168.56.10, destination ip: 192.168.56.10, port: 5000, protocol: tcp, time: 2017-02-22 13:21:22.824379, stdout: , stderr: nc: connect to 192.168.56.10 port 5000 (tcp) failed: Connection refused

)
failed: [centos7] => (item=source ip: 192.168.56.13, destination ip: 192.168.56.13, port: 443, protocol: tcp, time: 2017-02-22 13:21:23.574678, stdout: , stderr: Ncat: Version 6.40 ( http://nmap.org/ncat )
Ncat: Connection refused.

)
failed: [centos5] => (item=source ip: 192.168.56.10, destination ip: 192.168.56.13, port: 443, protocol: tcp, time: 2017-02-22 13:21:23.102524, stdout: , stderr: nc: connect to 192.168.56.13 port 443 (tcp) failed: No route to host

)

TASK [connectivity-test : debug] ***********************************************

PLAY RECAP *********************************************************************
centos5                    : ok=3    changed=0    unreachable=0    failed=1  
centos7                    : ok=3    changed=0    unreachable=0    failed=1  

```

## Manual way

```bash
dIP="192.168.56.10 192.168.56.13";
dPORT="5000";
sIP=$(/sbin/ip route get "$(echo $dIP|awk -F" " 'NR==1{print $1}')"|awk 'NR==1{print $NF}');
uname -n;
echo "Src IP: $sIP";
for i in $dIP;
do date;
nc -v -w 3 -s "$sIP" "$i" "$dPORT";
echo " ";
done
```
