def imageName="pioand/frontend"
def dockerTag=""
def dockerRegistry=""
def registryCredentials="dockerhub"

pipeline {
    agent {
      label 'agent'
    }
    environment {
        PIP_BREAK_SYSTEM_PACKAGES = 1
        scannerHome = tool 'SonarQube'
    }
    stages {
        
      stage('Download Code') {
        steps {
            checkout scm        
        }
        
        environment {
          BRANCH = "main"
        }        
      }
      
      stage('Unit Tests') {
        steps {
          sh "pip3 install -r requirements.txt"
          sh "python3 -m pytest --cov=. --cov-report xml:test-results/coverage.xml --junitxml=test-results/pytest-report.xml"            
        }
      }
      
      stage('Sonarqube analysis') {
        steps {
            withSonarQubeEnv('SonarQube') {
                sh "${scannerHome}/bin/sonar-scanner"                
            }
        }
      }

      stage('Docker image build') {
        steps {
            script {
                dockerTag = "RC-${env.BUILD_ID}.${env.GIT_COMMIT.take(7)}"
                applicationImage = docker.build("$imageName:$dockerTag", ".")
            }   
        }  
          
      }
      
      stage('Pushing image to Artifactory') {
        steps {
            script {
                docker.withRegistry("$dockerRegistry", "$registryCredentials") {
                    applicationImage.push()
                    applicationImage.push('latest')
                }   
            }
        }
      }
      
    }


    post {
        always {
            junit testResults: "test-results/*.xml"
            cleanWs()
        }
    }
}
