## Index

- [Docker and CentOS's firewalld service fix](DOCKER.md)
- [Enhance Email Security](EMAIL.md)
- [Prevent running process from randomly die](SWAP.md)

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

## Server Best Practices
- Use non-root user with sudo privileges and SSH
```
$ useradd [user]
$ passwd [user]
# Add [user] to the wheel group (sudo privileges)
$ gpasswd -a [user] wheel

# Login as the non-root user
$ su [user]


# Save the public key
$ mkdir ~/.ssh/
$ nano ~/.ssh/authorized_keys


# Change the folder and the file access permissions.
# chmod 700 = Only owner can read, write and execute.
# chmod 600 = Only owner can read and execute.
# Reference: https://www.thinkplexx.com/learn/article/unix/command/chmod-permissions-flags-explained-600-0600-700-777-100-etc
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
$ sudo timedatectl set-timezone UTC
$ sudo timedatectl set-local-rtc 0


# Restart the Rsyslog's service
$ sudo systemctl restart rsyslog
```

## NGINX Best Practices
Reference: https://docs.sucuri.net/warnings/hardening/

- Disable Server Banners by adding this code to the `/etc/nginx/nginx.conf` file

```
http {
         server_tokens off;
}
```

## SSL and TLS Deployment Best Practices
Reference:
- https://github.com/ssllabs/research/wiki/SSL-and-TLS-Deployment-Best-Practices
- https://ssllabs.com/ssltest to get the analysis report of the TLS configuration.

## PWA best practice: tips for designing great Progressive Web Apps
Reference:
- https://medium.com/progressivewebapps/pwa-best-practices-tips-for-designing-great-progressive-web-apps-96e92298d2ec
- https://chrome.google.com/webstore/detail/lighthouse/blipmdconlkpinefehnmjammfjpmpbjk?hl=en to get the analysis report of the web performance.

> Perceived Performance must be Seamless.
> Show placeholder and shadow first
> Make opacity transition.
