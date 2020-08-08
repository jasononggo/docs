# Docker Engine

## Install Docker Engine --nobest (3:18.09.1-3.el7) in CentOS 8

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

## Install Docker Compose 1.25.5

Reference: https://docs.docker.com/compose/release-notes/

```
$ sudo dnf install curl -y
$ sudo curl -L "https://github.com/docker/compose/releases/download/1.25.5/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
$ sudo chmod +x /usr/local/bin/docker-compose
$ sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```

# Enable SELinux

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

## SELinux Best Practices

Audit the `/var/log/audit/audit.log` with a dedicated security personnel.

You should not trust third party docker image, even the ones with open source repository.

## (general) Fix container did not work properly.

Reference: [Where to find SELinux permissions denials details](https://wiki.gentoo.org/wiki/SELinux/Tutorials/Where_to_find_SELinux_permission_denial_details)

- Disable SELinux to confirm whether SELinux is the culprit.

  ```
  # Set SELinux to permissive mode
  $ setenforce 0

  # After the test, set SELinux back to enforcing mode.
  $ setenforce 1
  ```

- Search for hidden denials

  ```
  # (optional) search SELinux policy dontaudit statements
  $ sesearch --dontaudit

  # disable `dontaudit` statements and rebuilds the SELinux policy. dev build only.
  $ semodule -DB

  # rebuilds the SELinux policy and enable `dontaudit` statement after deploying the services.
  $ semodule -B
  ```

- Search for the `AVC` logs

  ```
  # recent = 10 minutes ago
  $ ausearch -m AVC -ts recent | today
  ```

- [Use `audit2allow` to allow access.](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/security-enhanced_linux/sect-security-enhanced_linux-fixing_problems-allowing_access_audit2allow)

  Reference:
  - [SELinux policy for Containers](https://www.projectatomic.io/blog/2016/03/selinux-and-docker-part-2/)
  - [Create SELinux policy from `AVC` logs](https://stackoverflow.com/questions/52310241/how-to-modify-the-te-file-generated-by-audit2allow-and-recompile-it-into-pp-fi)
  - [Steps by steps to create SELinux policy from `AVC` logs](https://relativkreativ.at/articles/how-to-compile-a-selinux-policy-package)

  Make sure there is no unrelated AVC denial by running this command `$ ausearch -m AVC -ts recent`. If so, proceed.

  ```
  $ ausearch -m AVC -ts recent | audit2allow -M <policy_name>
  $ semodule -i <policy_name>.pp
  ```

- Enable iptables (legacy), avoid iptables (nf_table) and disable firewalld.

  Reference:
  - https://www.digitalocean.com/community/tutorials/how-to-migrate-from-firewalld-to-iptables-on-centos-7
  - https://www.tecmint.com/start-stop-disable-enable-firewalld-iptables-firewall/

  Most distributions use the `iptables` firewall, which uses the `netfilter` hooks to enforce firewall rules. CentOS 7 comes with an alternative service called `firewalld` which fulfills this same purpose.

## Fix for issue "docker container can write to the host's root directory"

Reference: https://dev.to/mkdev/hardening-docker-with-selinux-on-centos-8-4d6e

By default, Docker disable SELinux. To fix this, add this code to the `/etc/docker/daemon.json` file and run `$ systemctl restart docker`

```
{
  "selinux-enabled": true
}
```

## Fix for issue "docker container can not read into the host's root directory"

Reference:
- https://www.projectatomic.io/blog/2015/06/using-volumes-with-docker-can-cause-problems-with-selinux/
- https://danwalsh.livejournal.com/74095.html
- [Mounted volume is incorrectly labeled](https://www.projectatomic.io/blog/2016/07/docker-selinux-flag/)
- [What is the difference between the `z` and the `Z` flag in the docker volumes options?](https://stackoverflow.com/questions/35218194/what-is-z-flag-in-docker-containers-volumes-from-option)

Use the `z` and `Z` flag to the mounted volume instead of `ro` or `rw` flag.

> The z option indicates that the bind mount content is shared among multiple containers. The Z option indicates that the bind mount content is private and unshared. **This affects the file or directory on the host machine itself and can have consequences outside of the scope of Docker**.

## Fix for issue "docker no route to host and docker ACME challenge failed"

Reference: fix [issue #2719](https://github.com/fail2ban/fail2ban/issues/2719)

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

## Fix for issue [issue #41048](https://github.com/moby/moby/issues/41048)

Reference: https://www.linuxquestions.org/questions/linux-networking-3/iptables-v1-3-8-can%27t-initialize-iptables-table-%60filter%27-577212/

This issue happens if you run iptables in a container.

- Error output

  ```
  $ docker exec -ti proxy iptables -L
  modprobe: can't change directory to '/lib/modules': No such file or directory
  iptables v1.8.3 (legacy): can't initialize iptables table `filter': Table does not exist (do you need to insmod?)
  Perhaps iptables or your kernel needs to be upgraded.
  ```

- Check if the modules is loaded by running this command `$ lsmod | grep ip` in the host machine.

  ```
  ip_tables
  ip_conntrack
  iptable_filter
  ipt_state
  ```

- If not loaded, load the module by running this command `$ modprobe <module_name>` in the host machine.

  ```
  $ modprobe ip_tables
  $ modprobe ip_conntrack
  $ modprobe iptable_filter
  $ modprobe ipt_state
  ```

## Fix for container can't bind mounts to /var/log/secure.
  
  This is a steps by steps on how to create the SELinux policy. Use it to troubleshoot SELinux issue.
  
  - Summary
  
    You need to allow `container_t` to `read` and `open` to a file labeled with `var_log_t` type. 

    This is the generated `sample.te` file

    ```
    module sample 1.0;

    require {
            type var_log_t;
            type container_t;
            class file { open read };
    }

    #============= container_t ==============
    allow container_t var_log_t:file open;

    #!!!! This avc is allowed in the current policy
    allow container_t var_log_t:file read;
    ```

  - Build the policy package file (.pp)

    In order to build a policy package from a type enforcement file, we first have to convert it into a policy module. This is done with the checkmodule command:

    ```
    $ checkmodule -M -m -o sample.mod sample.te
    $ semodule_package -o sample.pp -m sample.mod
    $ semodule -i sample.pp
    ```
  
  - Bind mounts to /var/log/secure via `docker-compose` or `docker run`
  
    docker-compose.yml
  
    ```
    version: "3"

    services:
      fail2ban:
        image: crazymax/fail2ban
        volumes:
          - /var/log/secure:/var/log/secure:ro
    ```

    docker-compose

    ```
    $ docker-compose up -d
    $ docker exec -ti fail2ban cat /var/log/secure
    ```

    docker run

    ```
    $ docker run -it --rm -v /var/log/secure:/var/log/secure fail2ban cat /var/log/secure
    ```
  
  - Repeat this step until container can `$ cat /var/log/secure`
  
    Make sure there is no unrelated AVC denial by running this command `$ ausearch -m AVC -ts recent`. If so, proceed.
    
    ```
    $ ausearch -m AVC -ts recent | audit2allow -M <policy_name>
    $ semodule -i <policy_name>.pp
    ```

# iptables

Reference: [Docker and iptables](https://serverfault.com/questions/704643/steps-for-limiting-outside-connections-to-docker-container-with-iptables)

To avoid your rules being clobbered by `Docker`, use the `DOCKER-USER` chain.

> Conntrack supersedes state , but in modern kernels there is now no difference between the two. State is currently aliased and translated to conntrack in iptables if the kernel has it, so the syntax -m state --state is actually translated into -m conntrack --ctstate and handled by the same module.

You can use `-m conntrack --ctstate` instead of `-m state --state` if you want.

  - Block the `INPUT` chain and the `OUPUT` chain connection.
  
    ```
    # CHAIN INPUT
    # ---
    $ sudo iptables -I INPUT 1 -m state --state RELATED,ESTABLISHED -j ACCEPT
    $ sudo iptables -I INPUT 2 -m state --state NEW -p tcp --dport 22 -j ACCEPT
    $ sudo iptables -I INPUT 3 --reject-with icmp-host-prohibited -j REJECT


    # CHAIN OUTPUT
    # ---
    $ sudo iptables -I OUTPUT 1 -m state --state RELATED,ESTABLISHED -j ACCEPT
    
    # curl https
    $ sudo iptables -I OUTPUT 2 -p tcp -d example.com --dport 443 -m state --state NEW -j ACCEPT
    
    # dns lookup
    $ sudo iptables -I OUTPUT 3 -p udp --dport 53 -m state --state NEW -j ACCEPT

    $ sudo iptables -I OUTPUT 4 --reject-with icmp-host-prohibited -j REJECT
    ```

    Output:
    ```
    $ sudo iptables -L INPUT --line-numbers
    Chain INPUT (policy ACCEPT)
    num  target     prot opt source               destination
    1    ACCEPT     all  --  anywhere             anywhere             state RELATED,ESTABLISHED
    2    ACCEPT     tcp  --  anywhere             anywhere             state NEW tcp dpt:ssh
    3    REJECT     all  --  anywhere             anywhere             reject-with icmp-host-prohibited

    $ sudo iptables -L OUTPUT --line-numbers
    Chain OUTPUT (policy ACCEPT)
    num  target     prot opt source               destination
    1    ACCEPT     all  --  anywhere             anywhere             state RELATED,ESTABLISHED
    2    ACCEPT     udp  --  anywhere             anywhere             udp dpt:domain state NEW
    3    ACCEPT     tcp  --  anywhere             anywhere             tcp dpt:https state NEW
    4    REJECT     all  --  anywhere             anywhere             reject-with icmp-host-prohibited
    ```
  
  - Allow incoming SSH from specific IP address or subnet
  
    Reference: [iptables' essentials: common firewall rules and commands](https://www.digitalocean.com/community/tutorials/iptables-essentials-common-firewall-rules-and-commands#:~:text=Allow%20Incoming%20SSH%20from%20Specific,%2Dp%20tcp%20%2Ds%2015.15.)
  
    To allow incoming SSH connections from a specific IP address or subnet, specify the source. For example, if you want to allow the entire 15.15.15.0/24 subnet, run these commands:
  
    ```
    $ sudo iptables -I INPUT 2 -p tcp -s 15.15.15.0/24 --dport 22 -m state --state NEW -j ACCEPT
    ```

  - [Persistent iptables rules](https://www.thegeekdiary.com/centos-rhel-how-to-make-iptable-rules-persist-across-reboots/)
  
    ```
    $ service iptables save
    $ service iptables restart
    $ chkconfig iptables on
    ```

# Docker Swarm

## Disable firewalld and enable iptables

Reference: 
- https://www.digitalocean.com/community/tutorials/how-to-configure-the-linux-firewall-for-docker-swarm-on-centos-7
- https://gist.github.com/BretFisher/7233b7ecf14bc49eb47715bbeb2a2769

Most distributions use the `iptables` firewall, which uses the `netfilter` hooks to enforce firewall rules. CentOS 7 comes with an alternative service called `firewalld` which fulfills this same purpose.

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
  
## ASCII generator

  ```
  $ python3 -c "import base64, os; salt = base64.b64encode(os.urandom(12)); salted = base64.b64encode(base64.b64decode(salt) + os.urandom(12),b'__').decode(); print(salted)"
  ```
