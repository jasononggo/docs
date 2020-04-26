## Install Docker Engine and Docker Compose in Centos 8

### Install Docker Engine --nobest (3:18.09.1-3.el7) in CentOS 8 
Reference:
- https://docs.docker.com/engine/release-notes/
- https://linuxconfig.org/how-to-install-docker-in-rhel-8
```
sudo dnf config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo
sudo dnf list docker-ce
sudo dnf install docker-ce --nobest -y
sudo systemctl start docker
# Start Docker Engine at boot.
sudo systemctl enable docker
```

### Install Docker Compose 1.25.5
Reference: https://docs.docker.com/compose/release-notes/
```
sudo dnf install curl -y
sudo curl -L "https://github.com/docker/compose/releases/download/1.25.5/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```
