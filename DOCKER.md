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

### Docker and CentOS's firewalld service fix

Reference:
- https://www.digitalocean.com/community/tutorials/how-to-migrate-from-firewalld-to-iptables-on-centos-7
- https://www.tecmint.com/start-stop-disable-enable-firewalld-iptables-firewall/

Most distributions use the `iptables` firewall, which uses the `netfilter` hooks to enforce firewall rules. CentOS 7 comes with an alternative service called `firewalld` which fulfills this same purpose.

Usage:
- fix docker no route to host
- fix docker ACME challenge failed
- fix [issue #2719](https://github.com/fail2ban/fail2ban/issues/2719) with `$ sudo yum install iptables-services`

```
$ sudo systemctl stop firewalld
$ sudo systemctl disable firewalld
$ sudo systemctl mask firewalld
$ sudo yum install iptables-services
```
