pipeline {
    agent any
    tools {
        jdk 'jdk17'
        maven 'maven3'
    }
    environment {
        SCANNER_HOME= tool 'sonar-scanner'
    }    
    stages {
        stage('git checkout') {
            steps {
                git branch: 'main', credentialsId: 'git-cred', url: 'https://github.com/sanchitkedia03/Boardgame1.git'
            }
        }
         stage('compile') {
            steps {
              sh 'mvn compile'
            }
        }
         stage('test') {
            steps {
                sh 'mvn test'
            }
        }
         stage('file system scan') {
            steps {
               sh "trivy fs --format table -o trivy-fs-report.html"
            }
        }
         stage('sonarqube analysis') {
            steps {
                WithSonarQubeEnv('sonar'){
                    sh '''$SCANNER_HOME/bin/sonar-scanner -sonar.projectName=Boardgame -Dsonar.projectKey=Boardgame\
                    -Dsonar.java.binaries=. '''
                    
                }
            }
        }
         stage('Quality Gate') {
             steps{
                 script{
                     waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
                 }
             }
         }
          stage ("Build"){
              steps{
                  sh "mvn package"
              }
          }
          stage ("publish to nexus"){
              steps{
                  withMaven(globalMavenSettingConfig: 'global-settings', jdk: 'jdk17', maven:'maven3', traceability:true){
                     sh "mvn deploy"
                  }
              }
          }
          stage ("Build & tag docker image"){
              steps{
                  script{
                      withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker'){
                       sh "docker build -t sanchitkedia03/boardgame:latest ." 
                      }
                  }
              }
          }
           stage('docker image scan') {
             steps {
               sh "trivy image --format table -o trivy-fs-report.html sanchitkedia03/boardgame:latest"
             }  
          }
            stage ("push docker image"){
              steps{
                  script{
                      withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker'){
                       sh "docker push sanchitkedia03/boardgame:latest"
                      }
                  }      
              }
          }
          stage ("deploy to k8s"){
              steps{
                  withKubeConfig(caCertificate: '', clusterName: 'kubernetes', contextName: '', credentialsId: 'k8-cred' ){
                    sh "kubectl apply -f deployment-service.yaml"
                  }
              }
          }
           stage ("verify the deployment"){
              steps{
                  withKubeConfig(caCertificate: '', clusterName: 'kubernetes', contextName: '', credentialsId: 'k8-cred'){
                   sh "kubectl get pods -n webapps"
                   sh "kubectl get svc -n webapps"
                  }
              }
          }
          
    }
}
