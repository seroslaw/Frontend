def imageName="seroslaw/panda-frontend"
def dockerRegistry=""
def registryCredentials="Dockerhub"
def dockerTag=""

pipeline {
    
    agent {
      label 'agent'
    }
    
    environment {
        scannerHome = tool 'SonarQube'
    }
    
    stages {
        
        stage('Git Pull') {
            steps {
                //git branch: 'tests', credentialsId: 'artifactory', url: 'https://github.com/seroslaw/Frontend.git'
            	checkout scm
		}
        }
    
        stage('test') {
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
                timeout(time: 1, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }

            }
        }
        
        stage('Build application image') {
            steps {
                script {
                    //prepare basic image for app
                    dockerTag = "LoOoL"
                    
                    applicationImage = docker.build("$imageName:$dockerTag",".")
                }
            }
        }
        
        stage ('Pushing image to Artifactory') {
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

