# Multi-tier App Deployment with Jenkins Docker and Kubernetes Integration

This application has been developed by developer using the spring-boot java framework, the application needs a MongoDB database and we have used Jenkins, Docker and K8s for this project. 

This application has 3 different tiers;

- LOAD BALANCER Tier where endusers' traffic is routed to the application.
- APPLICATION Tier where endusers' traffic is received.
- DATABASE Tier where enduser's data from the application is stored.

# Build project using Maven
Maven is a java based build tool used to generate executable packages(jar, war, ear) for java based projects. The jenkinsfile performs a maven build with the command.
```
mvn clean package
```
Since we have Docker installed in the same server as Jenkins, we can add Jenkins user to the Docker group and build docker images by running the docker build command in the jenkinsfile.
```
docker build -t sekuns203/springapp .
```
We can equally deploy the docker images via the Jenkins server by installing Kubectl and running the the following command in the jenkinsfile.
```
kubectl apply -f springapp
```
*** take a look at the jenkinfile in this repo ***
 
# Multi-tier Infrastructure setup with AWS
 
# Step 1. Install and configure Docker and Jenkins on ubuntu 18 or higher
```
sudo apt install docker.io -y
sudo systemctl restart docker
sudo systemctl enable docker.service
```
Jenkins installation
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

 # Step 2. Install KOPS software on Ubuntu instance
```
sudo curl -LO https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64
chmod +x kops-linux-amd64 
sudo mv kops-linux-amd64 /usr/local/bin/kops
```

 # Step 3. Install Kubectl
```
sudo curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo chmod +x ./kubectl
sudo mv kubectl /usr/local/bin/ 
```

 # Step 4. Create an IAM role from AWS Console OR CLI with below policies
```
AmazonEC2FullAccess
AmazonRoute53FullAccess
AmazonS3FullAccess
IAMFullAccess
AmazonVPCFullAccess
AmazonSQSFullAccess
AmazonEventBridgeFullAccess
```

 # Step 5. Create S3 bucket and add environment variables in bashrc
```
aws s3 mb s3://<mybucket>
example: aws s3 mb s3://carasekuna.k8s.local
```
Add environment variables in bashrc:
```
vi .bashrc 
export NAME=mydeploy.k8s.local
export KOPS_STATE_STORE=s3://carasekuna.k8s.local
```
```
source .bashrc
```
 # Step 6. Create sshkeys before creating cluster
```
ssh-keygen 
```
 # Step 7. Create Kubernetes cluster definitions on S3 bucket
```
kops create cluster --zones us-east-2a --networking cilium --master-size t2.medium --master-count 1 --node-size t2.medium --node-count=2 ${NAME}

kops create secret --name ${NAME} sshpublickey admin -i ~/.ssh/id_rsa.pub
```
To list nodes
``` 
kubectl get nodes
```
To delete cluster
``` 
kops delete cluster --name=${NAME} --state=${KOPS_STATE_STORE} --yes
```
 # Step 8. Create Kubernetes cluster
```
kops update cluster ${NAME} --yes
```
```
kops validate cluster 
```
*** Take note of the suggested commands prompted on the terminal, you may have to wait 10 mins for the infrastructure to be fully provisioned ***
# Step 9. Access the application using AWS Elastic Load Balancer

Map the endpoint of the load balancer using route 53 service in AWS and create A record to allow endusers to easily access the application.

