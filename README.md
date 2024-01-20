
# Shopping App with DevSecOps

In this project we will use : Git, Github, Jenkins, SonarQube, Trivy, Owasp, K8s And Argocd


![App Screenshot](https://raw.githubusercontent.com/jeetu844/screenShots/main/Shopping-reactJS-DevOps/Jenkins_pipeline.png)
## Installation Steps

- Step1: [Jenkins Installation on CentOS 7](https://github.com/jeetu844/Shopping-reactJS-DevOps?tab=readme-ov-file#jenkins-installation)
- Step2: [Docker Installation on CentOS 7](https://github.com/jeetu844/Shopping-reactJS-DevOps?tab=readme-ov-file#docker-installation)
- Step3: [SonarQube Installation using Docker Image](https://github.com/jeetu844/Shopping-reactJS-DevOps?tab=readme-ov-file#sonarqube-installation)
- Step4: [Trivy Installation on CentOS 7](https://github.com/jeetu844/Shopping-reactJS-DevOps?tab=readme-ov-file#trivy-installation)
- Step5: [K8s Installation on CentOS 7](https://github.com/jeetu844/Shopping-reactJS-DevOps?tab=readme-ov-file#k8s-installation)
- Step6: [Argocd Installation on K8s](https://github.com/jeetu844/Shopping-reactJS-DevOps?tab=readme-ov-file#argocd-installation)
- Step7: [Prometheus Grafana Installation on K8s](https://github.com/jeetu844/Shopping-reactJS-DevOps#prometheus-grafana-installation-on-k8s)
## Configuration Steps
#### Jenkins Configuration
- [Jenkins Login](https://github.com/jeetu844/Shopping-reactJS-DevOps?tab=readme-ov-file#lets-jenkins-configuration)
- [Jenkins plugins installations](https://github.com/jeetu844/Shopping-reactJS-DevOps?tab=readme-ov-file#jenkins-plugins-installations)
- [Tools Configuration](https://github.com/jeetu844/Shopping-reactJS-DevOps?tab=readme-ov-file#jenkins-tools-installations)
#### SonarQube Configuration
- Create Token
- Create Webhook

## Jenkins Installation

Jenkins Installation on CentOS 7

```bash
yum update -y
sudo wget -O /etc/yum.repos.d/jenkins.repo \
    https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
sudo yum -y upgrade
sudo yum -y install java-11-openjdk
sudo yum -y install jenkins
sudo systemctl daemon-reload
service jenkins start
systemctl enable jenkins
```

## Docker Installation
Docker Installation on CentOS 7

```bash
yum install -y yum-utils device-mapper-persistent-data lvm2 vim git
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
yum install -y docker-ce docker-ce-cli containerd.io
systemctl enable docker
systemctl start docker
usermod -aG docker $USER
newgrp docker
```

## SonarQube Installation
SonarQube Installation using Docker Image

```bash
docker run -d -p 9000:9000 sonarqube:lts
```

## Trivy Installation 
Trivy Installation on CentOS 7

```bash
echo "[trivy]
name=Trivy repository
baseurl=https://aquasecurity.github.io/trivy-repo/rpm/releases/\$releasever/\$basearch/
gpgcheck=0
enabled=1" > /etc/yum.repos.d/trivy.repo

yum -y update
yum -y install trivy
```

## K8s Installation
K8s Installation on CentOS 7 **(For Master & Node)**

```bash
setenforce 0
sed -i --follow-symlinks 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux

cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF

yum install -y kubelet kubeadm kubectl
systemctl enable kubelet
systemctl start kubelet

sed -i '/swap/d' /etc/fstab
swapoff -a

cat <<EOF > /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sudo sysctl --system

rm -rf /etc/containerd/config.toml
systemctl restart containerd
```
**For K8s Master Only**
```bash
kubeadm init --pod-network-cidr=10.244.0.0/16

mkdir -p $HOME/.kube
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
chown $(id -u):$(id -g) $HOME/.kube/config

export KUBECONFIG=/etc/kubernetes/admin.conf

kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```
**For K8s Node Only**
- Replace this token with original which you get on master Node
```bash
kubeadm join 192.168.63.76:6443 --token u09eln.emqgd14u5p4wh2w0 \
--discovery-token-ca-cert-hash \
sha256:e5d568e17f8eda67c61b3f2addfcb74edc498a3b880c964b9a8072718a4e18ff
```
## Argocd Installation
Argocd Installation on K8s
```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/v2.4.7/manifests/install.yaml
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "NodePort"}}'
export ARGOCD_SERVER=`kubectl get svc argocd-server -n argocd -o json | jq --raw-output '.status.nodePort.ingress[0].hostname'`
export ARGO_PWD=`kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d`
```
- For get Argocd password 
```bash
echo $ARGO_PWD
```
## Prometheus Grafana Installation on K8s
Prometheus Grafana Installation on K8s using helm
```bash
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```
- See the Helm version
```bashh
helm version --client
```
```bash
helm repo add stable https://charts.helm.sh/stable
```
```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
```
```bash
kubectl create namespace prometheus
```
```bash
helm install stable prometheus-community/kube-prometheus-stack -n prometheus
```
```bash
kubectl get pods -n prometheus
```
```bash
kubectl get svc -n prometheus
```
To make Prometheus and grafana available outside the cluster, use LoadBalancer or NodePort instead of ClusterIP.
```bash
kubectl edit svc stable-kube-prometheus-sta-prometheus -n prometheus
```
![App Screenshot](https://raw.githubusercontent.com/jeetu844/screenShots/main/Shopping-reactJS-DevOps/k8s-prometheus.jpeg)
```bash
kubectl edit svc stable-grafana -n prometheus
```
```bash
kubectl get svc -n prometheus
```
### Let's Jenkins Configure
```bash
<Your Public IP Address:8080>
```
![Jenkins login password](https://raw.githubusercontent.com/jeetu844/screenShots/main/Jenkins/LoginPage.jpeg)

For get Administrator password
```bash
cat /var/lib/jenkins/secrets/initialAdminPassword
```
#### Now, install the suggested plugins.
![Suggested Plugins](https://raw.githubusercontent.com/jeetu844/screenShots/main/Jenkins/suggested-plugins.png)
#### Jenkins plugins installations
Goto Manage Jenkins -> Plugins -> Available plugins
- Eclipse Temurin Installer (Install without restart)
- SonarQube Scanner (Install without restart)
- NodeJs Plugin (Install Without restart)
- OWASP Dependency-Check (Install Without restart)
- Docker (Install Without restart)
- Docker Commons (Install Without restart)
- Docker Pipeline (Install Without restart)
- Docker API (Install Without restart)
- Docker-build-step (Install Without restart)

![Jenkins JDK Plugin Installation](https://raw.githubusercontent.com/jeetu844/screenShots/main/Jenkins/JDK-plugin.jpeg)

![Jenkins NodeJS Plugin Installation](https://raw.githubusercontent.com/jeetu844/screenShots/main/Jenkins/NodeJS-plugin.jpeg)

![OWASP Plugin Installation](https://raw.githubusercontent.com/jeetu844/screenShots/main/Jenkins/owasp.jpeg)

![Docker Plugin Installation](https://raw.githubusercontent.com/jeetu844/screenShots/main/Jenkins/docker.jpeg)

**Restart your Jenkins** ```<Your PublicIP:8080/restart>```

#### Jenkins Tools installations
Goto Manage Jenkins -> Tools
- ADD JDK
  - Name -> "jdk17"
  - Click on "Install automatically"
  - Add Installer "Install from adoptium.net"
  - Version "jdk-17.0.8.1+1"
![Jenkins JDK Configuration](https://raw.githubusercontent.com/jeetu844/screenShots/main/Jenkins/Jenkins-JDK-Configure.png)
- SonarQube Scanner installations
  - Click on "Add SonarQube Scanner"
  - Name "sonar-scanner"
![Jenkins Sonar Scanner Tool](https://raw.githubusercontent.com/jeetu844/screenShots/main/Jenkins/Jenkins-SonarTool-installation.png)
- NodeJS installations
  - Click on "Add NodeJS"
  - Name "nodejs20"
  - Click on "Install automatically"
  - Select "NodeJS 20.11.0"
![Jenkins NodeJS Tool](https://raw.githubusercontent.com/jeetu844/screenShots/main/Jenkins/Jenkins-NodeJS-Tool.png)
- Dependency-Check installations
  - Click on "Add Dependency-Check"
  - Name "DP-Check"
  - Click on "Install automatically"
  - Click on "Add Installer"
  - Select "Install from github.com"
  - Select Version "dependency-check 6.5.1"
![Jenkins DP-Check Tool](https://raw.githubusercontent.com/jeetu844/screenShots/main/Jenkins/Jenkins-DP-Check-Tool.png)
- Docker installations
  - Click on "Add Docker"
  - Name "docker"
  - Click on "Install automatically"
  - Click on "Add Installer"
  - Select "Download from docker.com"
  - Docker version "latest"
![Jenkins Docker Tool](https://raw.githubusercontent.com/jeetu844/screenShots/main/Jenkins/Jenkins-Docker-Tool.png)

### Let's SonarQube Configure
```
<Your Public IP Address:9000>
```
- Default Credentials
  - Username : admin
  - Password : admin
#### Create SonarQube Token
- Click on "Administration"
![SonarQube Step-1](https://raw.githubusercontent.com/jeetu844/screenShots/main/SonarQube/token-step-1.png)
- Click on "Security"
- Click on "Users"
![SonarQube Step-2](https://raw.githubusercontent.com/jeetu844/screenShots/main/SonarQube/token-step-2.png)
- Click on Update Tokens
![SonarQube Step-3](https://raw.githubusercontent.com/jeetu844/screenShots/main/SonarQube/token-step-3.png)
- Enter Token Name "Jenkins"
- Click on "Generate"
- Copy your Token
![SonarQube Step-4](https://raw.githubusercontent.com/jeetu844/screenShots/main/SonarQube/token-step-4.png)

## Authors

- [@Jitendra Sharma](https://www.github.com/jeetu844)
  