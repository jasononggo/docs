## Index

- [Docker and CentOS's firewalld service fix](DOCKER.md#centoss-firewalld-service-fix)
- [(optional) Docker and SELinux](DOCKER.md#optional-enable-selinux)
- [Enhance Email Security](EMAIL.md)
- [Prevent running process from randomly die](SWAP.md)
- [Web Server Best Practices](WEBSERVER.md)

## How To Write Proper Git Commit Message
Reference:
- https://medium.com/@steveamaza/how-to-write-a-proper-git-commit-message-e028865e5791
> Why is this change necessary?
> How does this commit address the issue?
> What effects does this change have?

## A template to make good README.md
Reference:
- https://gist.github.com/PurpleBooth/109311bb0361f32d87a2
- https://gist.github.com/PurpleBooth/b24679402957c63ec426 for details on our code of conduct, and the process for submitting pull requests to us.

## "Premature Optimization" and paranoid security measures

Reference: ["Premature Optimization" and paranoid security measures](https://stackoverflow.com/questions/12543607/prevent-session-cookie-hijacking-without-ssl/12545243#12545243)

> From my experience and from what I have read, I agree with the statement that "premature optimization is [one of the greatest evils of programming]". Focus your time on building a site that works and is useful for people and then, if something start to go wrong, think about getting an SSL certificate. You just don't have the money for the certificate and you don't have the money because your site has not yet been successful (so do that first).

> "Premature Optimization" and paranoid security measures are temptations for programmers because we like to think of ourselves as working on large-scale, very important projects when we are not [yet].

## Server Best Practices
- Use non-root user with sudo privileges and SSH
```
$ useradd [user]
$ passwd [user]


# Add [user] to the wheel group (sudo privileges)
# ---
$ gpasswd -a [user] wheel


# Login as the non-root user
# ---
$ su [user]


# Save the public key
# ---
$ mkdir ~/.ssh/
$ nano ~/.ssh/authorized_keys

# Change the folder and the file access permissions.
# chmod 700 = Only owner can read, write and execute.
# chmod 600 = Only owner can read and execute.
# Reference: https://www.thinkplexx.com/learn/article/unix/command/chmod-permissions-flags-explained-600-0600-700-777-100-etc
# ---
$ chmod 700 ~/.ssh/
$ chmod 600 ~/.ssh/authorized_keys
```

- Disable Root Login
```
$ sudo sed -i '/^PermitRootLogin/s/yes/no/' /etc/ssh/sshd_config
$ sudo systemctl reload sshd
```

- Disable Password Based Login
```
$ sudo sed -i '/^ChallengeResponseAuthentication/s/yes/no/' /etc/ssh/sshd_config
$ sudo sed -i '/^PasswordAuthentication/s/yes/no/' /etc/ssh/sshd_config
$ sudo sed -i '/^UsePAM/s/yes/no/' /etc/ssh/sshd_config
$ sudo systemctl reload sshd
```

- Set the timezone to UTC
```
# Set the timezone to UTC
# ---
$ sudo timedatectl set-timezone UTC
$ sudo timedatectl set-local-rtc 0


# Restart the Rsyslog's service
# ---
$ sudo systemctl restart rsyslog
```
