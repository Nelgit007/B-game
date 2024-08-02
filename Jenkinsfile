pipeline {    
    agent any

    environment {
        registry = "nelsonosagie/nelapp"
        registryCredential = 'docker-cred'
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

        // stage('Compile') {
        //     steps {
        //         sh 'mvn compile'
        //     }
        // }
        
        // stage('Test') {
        //     steps {
        //         sh 'mvn test'
        //     }
        // }
        
        // stage('Build') {
        //     steps {
        //         sh 'mvn package'
        //     }
        // }
    }
}