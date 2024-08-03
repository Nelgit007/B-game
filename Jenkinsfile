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

        stage('Docker Image Build') {
          steps {
            script {
                sh "docker build -t ${JOB}:V${BUILD_NUMBER} ."
                
                // withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                //     sh "docker build -t ${JOB}:V${BUILD_NUMBER} ."
                // }
            }
          }
        }

        // stage('Docker Image Scan') {
        //     steps {
        //         sh "trivy image --format table -o trivy-fs-report.html ${JOB}:V${BUILD_NUMBER}"
        //     }
        // }

        stage('Docker Image Tag'){
          steps{
            script {
                def COMMIT_ID = env.GIT_COMMIT.take(7)
                sh "docker tag ${JOB}:V${BUILD_NUMBER} ${DOCKER_USERNAME}/${JOB}:V${BUILD_NUMBER}"
                sh "docker tag ${JOB}:V${BUILD_NUMBER} ${DOCKER_USERNAME}/${JOB}:${COMMIT_ID}"

                
            //   docker.withRegistry('', registryCredential) {
            //     dockerImage.push("${DOCKER_USERNAME}/${JOB}:V${BUILD_NUMBER}")
            //     dockerImage.push("${DOCKER_USERNAME}/${JOB}:latest")
            //   }
            }
          }
        }

        stage('Dockerhub login') {
            steps {
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
                echo 'Login Successful'
            }
        }

        stage('Docker Image Push') {
            steps {
                script {
                    def COMMIT_ID = env.GIT_COMMIT.take(7)
                    //sh "docker push ${DOCKER_USERNAME}/${JOB}:v${BUILD_NUMBER}"
                    sh "docker push ${DOCKER_USERNAME}/${JOB}:V${BUILD_NUMBER}"
                    sh "docker push ${DOCKER_USERNAME}/${JOB}:${COMMIT_ID}"
                }
            }
        }

        stage('Deploy to K8s') {
          steps {
            script {
                withKubeCredentials(kubectlCredentials: [[caCertificate: '', clusterName: 'kopsblog.com.ng', contextName: '', credentialsId: 'k8-cred', namespace: 'webapps', serverUrl: 'https://api.kopsblog.com.ng']]) {
                    sh "kubectl apply -f deployment-service.yaml"
                }
            }
          }
        }

        stage('Verify Deployment to K8s') {
          steps {
            script {
                withKubeCredentials(kubectlCredentials: [[caCertificate: '', clusterName: 'kopsblog.com.ng', contextName: '', credentialsId: 'k8-cred', namespace: 'webapps', serverUrl: ' https://api.kopsblog.com.ng']]) {
                    sh "kubectl get pods -n webapps"
                    sh "kubectl get svc -n webapps"
                }
            }
          }
        }

        stage('Docker Image clean up') {
            steps {
                sh(script: 'docker image prune -af')
            }
        }
    }

    post {
    always {
        script {
            def jobName = env.JOB_NAME
            def buildNumber = env.BUILD_NUMBER
            def pipelineStatus = currentBuild.result ?: 'UNKNOWN'
            def bannerColor = pipelineStatus.toUpperCase() == 'SUCCESS' ? 'green' : 'red'

            def body = """
                <html>
                <body>
                <div style="border: 4px solid ${bannerColor}; padding: 10px;">
                <h2>${jobName} - Build ${buildNumber}</h2>
                <div style="background-color: ${bannerColor}; padding: 10px;">
                <h3 style="color: white;">Pipeline Status: ${pipelineStatus.toUpperCase()}</h3>
                </div>
                <p>Check the <a href="${BUILD_URL}">console output</a>.</p>
                </div>
                </body>
                </html>
            """

            emailext (
                subject: "${jobName} - Build ${buildNumber} - ${pipelineStatus.toUpperCase()}",
                body: body,
                to: 'osagienelson24@gmail.com',
                from: 'jenkins@example.com',
                replyTo: 'jenkins@example.com',
                mimeType: 'text/html'
                // attachmentsPattern: 'trivy-image-report.html'
            )
        }
    }
}
}