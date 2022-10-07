## 1. Installation docker on AWS ubuntu
- Set up the repository
```bash
sudo apt-get update
sudo apt-get install -y ca-certificates curl gnupg lsb-release
```
- Add Docker’s official GPG key:
```bash
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

- Use the following command to set up the repository:
```bash
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
- Install Docker Engine
```bash
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin
```
## 2. Install Rancher
```bash
sudo docker run -d --restart=unless-stopped -p 80:80 -p 443:443 --privileged rancher/rancher:v2.6.8
```
```bash
# get container id
docker ps
# get rancher default password
docker logs  container-id  2>&1 | grep "Bootstrap Password:"
```