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
       buildDiscarder(logRotator(numToKeepStr: '10')) 
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
                sh 'mvn clean package'
            }
        }
        stage('Building container image') {
            steps{
                sh "docker build -t $IMAGE_NAME:$IMAGE_TAG ."
                sh "docker build -t $IMAGE_NAME:latest ."
            }
        }
        stage('Docker login'){
            steps{
                withCredentials([string(credentialsId: 'dockerpassword', variable: 'dockerpasswd')]) {
                    sh "docker login -u sanjeetkr -p $dockerpasswd"
                }
            }
        }
         stage('Push to dockerhub'){
            steps{
                sh "docker push $IMAGE_NAME:$IMAGE_TAG"
                sh "docker push $IMAGE_NAME:latest"
            }
        }
        stage('Run as container'){
            steps{
                sh "docker rm -f web || true"
                sh "docker run -d -p 80:8060 --name web $IMAGE_NAME:$IMAGE_TAG"
            }
        }
        stage ('Deploy to tomcat') {
            //when {
                //expression { false }
           // }
            steps {
                withCredentials([string(credentialsId: 'tomcat_pd', variable: 'tomcat_pwd')]) {
                    sh "curl --user tomcat:$tomcat_pwd --upload-file $WAR_PATH 'http://$TOMCAT_HOST:$TOMCAT_PORT/manager/text/deploy?path=/$CONTEXT_NAME&update=true'"
                
                    // curl --user <tomcatuser>:<tomcatpwd> --upload-file <warpath>
                   // 'http://tomcat_ip:port/manager/text/deploy?path=/<context>&update=true'
                }
            }
        }

    }
}