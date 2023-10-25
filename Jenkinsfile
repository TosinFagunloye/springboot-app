pipeline {
   tools {
       maven 'Maven3'
   }
   agent any
   environment {
    registry = "883146812401.dkr.ecr.us-east-1.amazonaws.com/anmute_project_jenkins:latest"
   }
   stages {
       stage('Cloning Git') {
           steps {
               checkout([$class: 'GitSCM', branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: '', url: 'https://github.com/TosinFagunloye/springboot-app.git']]])
           }
       }
       stage('Build and Scan Jar File') {
           steps {
               withSonarQubeEnv(installationName: 'sonarqubescanner', credentialsId: '834df684-0a63-43af-8f87-b3cf5c592e2b') {
               sh 'mvn clean install sonar:sonar'
               }
           }
       }
       stage('Docker Image Build') {
           steps {
               sh 'docker build -t anmute_project_jenkins .'
           }
       }
       stage('Push Docker Image to ECR') {
           steps {
               withAWS(credentials: '1ed24dac-56c1-4a35-9999-e37f189cc5fe', region: 'us-east-1') {
                   sh 'aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 883146812401.dkr.ecr.us-east-1.amazonaws.com'
                   sh 'docker tag anmute_project_jenkins:latest 883146812401.dkr.ecr.us-east-1.amazonaws.com/anmute_project_jenkins:latest'
                   sh 'docker push 883146812401.dkr.ecr.us-east-1.amazonaws.com/anmute_project_jenkins:latest'
               }
           }
       }
       stage('Integrate Jenkins with EKS Cluster and Deploy App') {
           steps {
               withAWS(credentials: '1ed24dac-56c1-4a35-9999-e37f189cc5fe', region: 'us-east-1') {
                 script {
                   sh ('aws eks update-kubeconfig --name prod --region us-east-1')
                   sh ('kubectl apply -f eks-deploy-k8s.yaml')
               }
               }
       }
   }
   }
}
