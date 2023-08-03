pipeline {
    agent any
    
    tools {
        maven 'maven3'
    }
    environment{
        SCANNER_HOME= tool 'sonar-scanner'
    }

    stages {

        stage('GIT CHECKOUT') {
            steps {
                git branch: 'main', changelog: false, poll: false, url: 'https://github.com/SyedWasifAbbas5/java-app-cicd.git'
            }
        }
        
        stage('CODE COMPILE') {
            steps {
               sh "mvn clean compile"
            }
        }
        
        stage('SONARQUBE ANALYSIS') {
            steps {
               withSonarQubeEnv('sonar-scanner') {
                  sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=java-app-cicd \
                  -Dsonar.java.binaries=. \
                  -Dsonar.projectKey= java-app-cicd '''
                }
            }
        }

		stage('TRIVY SCAN') {
            steps {
               sh " trivy fs --securtiy-checks vuln,config /var/lib/jenkins/workspace/CICD "
            }
        }
		
		stage('CODE BUILD') {
            steps {
               sh " mvn clean install "
            }
        }

           stage("Build and Test"){
            steps{
                sh "docker build . -t java-app-cicd"
            }
        }
        
        stage("Push to Docker Hub"){
            steps{
                script{
                    withDockerRegistry(credentialsId: '527463dd-cc1c-4a48-884a-f25b4db7d023') {
                        sh "docker tag java-app-cicd syedwasifabbas/java-app-cicd:$BUILD_ID"
                        sh "docker push syedwasifabbas/java-app-cicd:$BUILD_ID"
                  }
                }
            }
        }

        stage("DEPLOY TO K8s"){
            steps{
                 script{
                    kubernetesDeploy(configs: 'deploymentservice.yaml',  kubeconfigId: 'kubernetes')
                }
            }
        }
    }             
}
