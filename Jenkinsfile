node
{
def mavenHome = tool name: 'maven3.8.7'
    stage('SCM Clone') 
    {
    git credentialsId: 'Github-Cred', url: 'https://github.com/jerrysekuna/spring-boot-docker.git'
    }
    stage('Maven Build'){
        sh "${mavenHome}/bin/mvn clean package"
    }
    stage('Quality Report') {
        //sh '${mavenHome}/bin/sonar:sonar'
    }
    stage('Upload Artifacts') {
        //sh '${mavenHome}/bin/mvn deploy'
    }
    stage('Build Docker Image'){
        sh "docker build -t sekuns203/spring-boot-app ."
    }
    stage('Push Image Dockerhub'){
        withCredentials([string(credentialsId: 'Dockerhub-Cred', variable: 'Docker-Cred')]) {
    //sh "docker login -u sekuns203 -p ${Docker-Cred}"
    'cat ~/my_password.txt | docker login --username sekuns203 --password-stdin'
        }
    'sh "docker push sekuns203/spring-boot-app:latest'
    }
    stage('Remove Docker Images')
    {
      sh 'docker rmi $(docker images -q)'
    }
    stage('Deploy to K8s'){
        
        sh 'kubectl apply -f springapp.yml'
    }
} 