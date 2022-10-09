pipeline {
    agent any
    environment {
        WAR_PATH="$WORKSPACE/target/*.war"
        TOMCAT_HOST="10.0.0.141"
        TOMCAT_PORT="8090"
        CONTEXT_NAME="demowebapp"
        IMAGE_NAME="sanjeetkr/emp_webapp"
        IMAGE_TAG="$BUILD_NUMBER"
    }
    options {
       buildDiscarder(logRotator(numToKeepStr: '2')) 
       timeout(time: 5, unit: 'MINUTES')
       disableConcurrentBuilds()
       quietPeriod(5)
       retry(3)
    }
    tools {
        maven 'Maven_local' 
    }
    stages {
        stage('Cleanup Workspace') {
            steps {
                script {
                    cleanWs()
                    sh "echo Cleaned Up Workspace for ${JOB_NAME}"
                }
            }
        }
        stage('SCM Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/sanjeetcalgary/Employee_webapp.git'
            }
        }
        stage('Generate the artifact') {
            steps {
                sh 'mvn -Dmaven.test.failure.ignore=true clean package'
            }
        }
        stage('Building container image') {
            steps{
                sh "docker build -t $IMAGE_NAME:$IMAGE_TAG ."
                sh "docker build -t $IMAGE_NAME:latest ."
            }
        }       
    }
}