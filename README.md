# Multi-tier App Deployment with Jenkins Docker and Kubernetes Integration

 # Install and configure Docker on ubuntu 18 or higher in AWS
```
sudo apt install docker.io -y
sudo systemctl restart docker
sudo systemctl enable docker.service
```

 # Install, start and configure Jenkins same ubuntu instance
```
sudo wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add - 
sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > \/etc/apt/sources.list.d/jenkins.list'
sudo apt updata && sudo apt install jenkins
sudo systemctl start jenkins && sudo systemctl status jenkins
sudo usermod -aG docker jenkins
systemctl enable docker.service
sudo echo "jenkins ALL=(ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/jenkins
```

 *** setup Kubernetes cluster on AWS using KOPS ***

 # Install KOPS software on Ubuntu instance
```
sudo curl -LO https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64
chmod +x kops-linux-amd64 
sudo mv kops-linux-amd64 /usr/local/bin/kops
```

 # Install Kubectl
```
sudo curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo chmod +x ./kubectl
sudo mv kubectl /usr/local/bin/ 
```

 # Create an IAM role from AWS Console OR CLI with below policies
```
AmazonEC2FullAccess
AmazonRoute53FullAccess
AmazonS3FullAccess
IAMFullAccess
AmazonVPCFullAccess
AmazonSQSFullAccess
AmazonEventBridgeFullAccess
```

 # Create S3 bucket 
```
aws s3 mb s3://<mybucket>
example: aws s3 mb s3://carasekuna.k8s.local
```

 # Add environment variables in bashrc:

```
vi .bashrc 
export NAME=mydeploy.k8s.local
export KOPS_STATE_STORE=s3://carasekuna.k8s.local
```
 # Create sshkeys before creating cluster
```
ssh-keygen 
```
 # Create Kubernetes cluster definitions on S3 bucket
```
kops create cluster --zones us-east-2a --networking cilium --master-size t2.medium --master-count 1 --node-size t2.medium --node-count=2 ${NAME}

kops create secret --name ${NAME} sshpublickey admin -i ~/.ssh/id_rsa.pub
```
``` To list nodes
kubectl get nodes
```

``` To delete cluster
kops delete cluster --name=${NAME} --state=${KOPS_STATE_STORE} --yes
```

 # Create Kubernetes cluster
```
kops update cluster ${NAME} --yes
```
*** Take note of the suggested commands prompted on the terminal ***

```
kops validate cluster
```# springapp-deployment
# springapp-deployment
