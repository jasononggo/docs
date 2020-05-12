## Install Docker Engine and Docker Compose in Centos 8

### Install Docker Engine --nobest (3:18.09.1-3.el7) in CentOS 8 
Reference:
- https://docs.docker.com/engine/release-notes/
- https://linuxconfig.org/how-to-install-docker-in-rhel-8
```
$ sudo dnf config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo
$ dnf list docker-ce
$ sudo dnf install docker-ce --nobest -y
$ sudo systemctl start docker


# Start Docker Engine at boot.
$ sudo systemctl enable docker
```

### Install Docker Compose 1.25.5
Reference: https://docs.docker.com/compose/release-notes/
```
$ sudo dnf install curl -y
$ sudo curl -L "https://github.com/docker/compose/releases/download/1.25.5/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
$ sudo chmod +x /usr/local/bin/docker-compose
$ sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```

### Disable CentOS's firewalld service

Keywords:
- fix docker no route to host
- fix docker ACME challenge failed

Reference:
- https://www.digitalocean.com/community/tutorials/how-to-migrate-from-firewalld-to-iptables-on-centos-7
- https://www.tecmint.com/start-stop-disable-enable-firewalld-iptables-firewall/

Most distributions use the `iptables` firewall, which uses the `netfilter` hooks to enforce firewall rules. CentOS 7 comes with an alternative service called `firewalld` which fulfills this same purpose.

```
$ sudo systemctl stop firewalld
$ sudo systemctl disable firewalld
$ sudo systemctl mask firewalld
```

fix [issue #2719](https://github.com/fail2ban/fail2ban/issues/2719) with `$ sudo yum install iptables-services`

```
fail2ban-input    | 2020-05-11 23:25:15,825 fail2ban.actions        [1]: NOTICE  [sshd] Ban 195.231.4.203
fail2ban-input    | 2020-05-11 23:25:15,836 fail2ban.utils          [1]: ERROR   7fb207b63240 -- exec: iptables -w -N f2b-sshd
fail2ban-input    | iptables -w -A f2b-sshd -j RETURN
fail2ban-input    | iptables -w -I INPUT -p tcp -m multiport --dports ssh -j f2b-sshd
fail2ban-input    | 2020-05-11 23:25:15,836 fail2ban.utils          [1]: ERROR   7fb207b63240 -- stderr: 'modprobe: FATAL: Module ip_tables not found in directory /lib/modules/4.18.0-147.5.1.el8_1.x86_64'
fail2ban-input    | 2020-05-11 23:25:15,836 fail2ban.utils          [1]: ERROR   7fb207b63240 -- stderr: "iptables v1.8.3 (legacy): can't initialize iptables table `filter': Table does not exist (do you need to insmod?)"
fail2ban-input    | 2020-05-11 23:25:15,836 fail2ban.utils          [1]: ERROR   7fb207b63240 -- stderr: 'Perhaps iptables or your kernel needs to be upgraded.'
fail2ban-input    | 2020-05-11 23:25:15,836 fail2ban.utils          [1]: ERROR   7fb207b63240 -- stderr: 'modprobe: FATAL: Module ip_tables not found in directory /lib/modules/4.18.0-147.5.1.el8_1.x86_64'
fail2ban-input    | 2020-05-11 23:25:15,836 fail2ban.utils          [1]: ERROR   7fb207b63240 -- stderr: "iptables v1.8.3 (legacy): can't initialize iptables table `filter': Table does not exist (do you need to insmod?)"
fail2ban-input    | 2020-05-11 23:25:15,836 fail2ban.utils          [1]: ERROR   7fb207b63240 -- stderr: 'Perhaps iptables or your kernel needs to be upgraded.'
fail2ban-input    | 2020-05-11 23:25:15,836 fail2ban.utils          [1]: ERROR   7fb207b63240 -- stderr: 'modprobe: FATAL: Module ip_tables not found in directory /lib/modules/4.18.0-147.5.1.el8_1.x86_64'
fail2ban-input    | 2020-05-11 23:25:15,836 fail2ban.utils          [1]: ERROR   7fb207b63240 -- stderr: "iptables v1.8.3 (legacy): can't initialize iptables table `filter': Table does not exist (do you need to insmod?)"
fail2ban-input    | 2020-05-11 23:25:15,836 fail2ban.utils          [1]: ERROR   7fb207b63240 -- stderr: 'Perhaps iptables or your kernel needs to be upgraded.'
fail2ban-input    | 2020-05-11 23:25:15,836 fail2ban.utils          [1]: ERROR   7fb207b63240 -- returned 3
fail2ban-input    | 2020-05-11 23:25:15,837 fail2ban.actions        [1]: ERROR   Failed to execute ban jail 'sshd' action 'iptables-multiport' info 'ActionInfo({'ip': '195.231.4.203', 'family': 'inet4', 'fid': <function Actions.ActionInfo.<lambda> at 0x7fb20793cdc0>, 'raw-ticket': <function Actions.ActionInfo.<lambda> at 0x7fb20793d4c0>})': Error starting action Jail('sshd')/iptables-multiport: 'Script error'
```
