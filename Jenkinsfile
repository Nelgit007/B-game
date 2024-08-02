pipeline {    
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('docker-cred')
        registryCredential = "docker-cred"
        DOCKER_USERNAME = "nelsonosagie"
        JOB = "bgame"
        SCANNER_HOME = tool 'mysonarscanner4'
    }

    tools {
        jdk 'OracleJDK17'
        maven '3.8.1'
    }

    stages {   
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Clone Repository'){
             steps{
             git branch: 'main', url: 'https://github.com/Nelgit007/B-game.git'
            }
        }

        stage('Compile') {
            steps {
                sh 'mvn compile'
            }
        }
        
        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }
        
        stage('File System Scan') {
            steps {
                sh "trivy fs --format table -o trivy-fs-report.html ."
            }
        }

        // stage('SonarQube Analysis') {
        //     steps {
        //         withSonarQubeEnv('sonar-scan') {
        //             sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName-Boardgame -Dsonar.projectKey-Boardgame \
        //             -Dsonar.java.binaries-.'''
        //         }
        //     }
        // }

        // stage('Quality Gate') {
        //   steps {
        //     script {
        //         waitForQualityGate abortPipeline: false, credentialsId: 'sonar-scan'
        //     }
        //   }
        // }

        stage('Build App') {
          steps {
            script {
                sh 'mvn package'
            }
          }
        }

        stage('Build & Tag Docker Image') {
          steps {
            script {
                withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                    sh "docker build -t ${JOB}:V${BUILD_NUMBER} ."
                }
            }
          }
        }

        // stage('Docker Image Scan') {
        //     steps {
        //         sh "trivy image --format table -o trivy-fs-report.html ${JOB}:V${BUILD_NUMBER}"
        //     }
        // }

        stage('Upload Image'){
          steps{
            script {
              docker.withRegistry('', registryCredential) {
                dockerImage.push("${DOCKER_USERNAME}/${JOB}:V${BUILD_NUMBER}")
                dockerImage.push("${DOCKER_USERNAME}/${JOB}:latest")
              }
            }
          }
        }

        stage('Remove Unused docker image') {
          steps{
            sh "docker rmi ${JOB}:${BUILD_NUMBER} ${DOCKER_USERNAME}/${JOB}:V${BUILD_NUMBER}"
            sh "docker rmi ${JOB}:${BUILD_NUMBER} ${DOCKER_USERNAME}/${JOB}:latest"
          }
        }
    }
}