# Docker Engine

## Install Docker Engine and Docker Compose in Centos 8

### Install Docker Engine --nobest (3:18.09.1-3.el7) in CentOS 8 
Reference:
- https://docs.docker.com/engine/release-notes/
- https://linuxconfig.org/how-to-install-docker-in-rhel-8
```
$ sudo dnf config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo
$ sudo dnf install docker-ce --nobest -y
$ sudo systemctl start docker


# Start Docker Engine at boot.
# ---
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

## CentOS's firewalld service fix

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
$ sudo systemctl start iptables


# Start iptables at boot
$ sudo systemctl enable iptables

$ sudo systemctl restart docker
```

## (optional) Enable SELinux

Reference: - https://www.alibabacloud.com/blog/selinux-usage-in-alibaba-cloud-ecs_594556

- Replace `SELINUX=disabled` to `SELINUX=enforcing` in the `/etc/selinux/config` file

- Tell SELinux to relabel all of the files on that system with the correct SELinux contexts **on the next boot**. On large disks, this process can take a good amount of time.

```
$ su root
$ touch /.autorelabel
$ shutdown -r now
```

- Once logged in, Verify that SELinux is activated

```
$ getenforce
$ sestatus
```

### (optional) Fix for issue "docker container can write to the host's root directory"

Reference: - https://dev.to/mkdev/hardening-docker-with-selinux-on-centos-8-4d6e

By default, Docker disable SELinux. To fix this, add this code to the `/etc/docker/daemon.json` file and run `$ systemctl restart docker`

```
{
  "selinux-enabled": true
}
```

#### Fix for issue "docker container can not read into the host's root directory"

Reference:
- https://www.projectatomic.io/blog/2015/06/using-volumes-with-docker-can-cause-problems-with-selinux/
- https://danwalsh.livejournal.com/74095.html
- [Mounted volume is incorrectly labeled](https://www.projectatomic.io/blog/2016/07/docker-selinux-flag/)
- [What is the difference between the `z` and the `Z` flag in the docker volumes options?](https://stackoverflow.com/questions/35218194/what-is-z-flag-in-docker-containers-volumes-from-option)

Use the `z` and `Z` flag to the mounted volume instead of `ro` or `rw` flag.

> The z option indicates that the bind mount content is shared among multiple containers. The Z option indicates that the bind mount content is private and unshared. **This affects the file or directory on the host machine itself and can have consequences outside of the scope of Docker**.

# Docker Swarm

## Disable firewalld and enable iptables

Reference: 
- https://www.digitalocean.com/community/tutorials/how-to-configure-the-linux-firewall-for-docker-swarm-on-centos-7
- https://gist.github.com/BretFisher/7233b7ecf14bc49eb47715bbeb2a2769

### Steps

- Enable iptables on each node

```
$ sudo systemctl stop firewalld
$ sudo systemctl disable firewalld
$ sudo systemctl mask firewalld
$ sudo yum install iptables-services
$ sudo systemctl start iptables
$ sudo systemctl enable iptables
```

- (caution) This first set of commands should be executed on the nodes that will serve as **Swarm managers**.

```
# Inbound to Swarm Managers
# ---
$ sudo iptables -I INPUT 5 -p tcp --dport 2377 -j ACCEPT
$ sudo iptables -I INPUT 6 -p tcp --dport 7946 -j ACCEPT
$ sudo iptables -I INPUT 7 -p udp --dport 7946 -j ACCEPT
$ sudo iptables -I INPUT 8 -p udp --dport 4789 -j ACCEPT

# Those rules are runtime rules and will be lost if the system is rebooted.
# To save the current runtime rules to a file so that they persist after a reboot, type:
# ---
$ sudo /usr/libexec/iptables/iptables.init save

$ systemctl restart docker
```

- (caution) This first set of commands should be executed on the nodes that will serve as **Swarm workers**.

```
# Inbound to Swarm workers
# ---
$ sudo iptables -I INPUT 5 -p tcp --dport 7946 -j ACCEPT
$ sudo iptables -I INPUT 6 -p udp --dport 7946 -j ACCEPT
$ sudo iptables -I INPUT 7 -p udp --dport 4789 -j ACCEPT

$ sudo /usr/libexec/iptables/iptables.init save
$ sudo systemctl restart docker
```

- (optional) If you need to access the Docker daemon remotely, you need to enable the `tcp` Socket, use port `2376` for encrypted communication with the daemon.

```
$ sudo iptables -I INPUT 5 -p tcp --dport 2376 -j ACCEPT
```
