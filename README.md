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

## SSL and TLS Deployment Best Practices
Reference:
- https://github.com/ssllabs/research/wiki/SSL-and-TLS-Deployment-Best-Practices
- https://ssllabs.com/ssltest to get the analysis report of the TLS configuration.

## Docker Best Practice
- Use non-root user with sudo privileges
```
useradd [user]
passwd [user]
# Add [user] to the wheel group (sudo privileges)
gpasswd -a [user] wheel
```


- Set the timezone to UTC
```
# Set the timezone to UTC
sudo timedatectl set-timezone UTC
sudo timedatectl set-local-rtc 0
# Restart the Rsyslog's service
sudo systemctl restart rsyslog


# Install Docker Engine --nobest (3:18.09.1-3.el7) in CentOS 8 
Reference: https://docs.docker.com/engine/release-notes/
Reference: https://linuxconfig.org/how-to-install-docker-in-rhel-8
sudo dnf config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo
sudo dnf list docker-ce
sudo dnf install docker-ce --nobest -y
sudo systemctl start docker
# Start Docker Engine at boot.
sudo systemctl enable docker


# Install Docker Compose 1.25.5
Reference: https://docs.docker.com/compose/release-notes/
sudo dnf install curl -y
sudo curl -L "https://github.com/docker/compose/releases/download/1.25.5/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```
