# OpenED

```
sudo dnf install wget curl nano unzip yum-utils -y
sudo firewall-cmd --state
sudo firewall-cmd --permanent --list-services
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --permanent --add-masquerade
 sudo firewall-cmd --reload
 sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
    sudo dnf install docker-ce docker-ce-cli containerd.io
    ```
    
