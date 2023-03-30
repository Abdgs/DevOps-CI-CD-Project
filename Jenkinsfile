pipeline {
    agent any

    environment {
        DOCKER_HUB = credentials('DOCKER_HUB')
        EC2_CREDENTIALS = credentials('EC2')
    }

    stages {
        stage('Checkout') {
            steps {
                checkout([$class: 'GitSCM',
                          branches: [[name: '*/main']],
                          userRemoteConfigs: [[url: 'https://github.com/Abdgs/DevOps-CI-CD-Project.git']]])
            }
        }
        stage('Build') {
            steps {
                sh 'mvn clean package'
                docker build -t my-image .
            }
        }
        stage('Test') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }
        stage('Push to Docker Hub') {
            steps {
                withCredentials([string(credentialsId: 'DOCKER_HUB', variable: 'DOCKER_HUB')]) {
                    sh 'docker login -u $DOCKER_HUB_USR -p $DOCKER_HUB_PSW'
                    sh 'docker push my-image'
                }
            }
        }
        stage('Deploy to EC2') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'EC2', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY']]) {
                    sshagent(['EC2']) {
                        sh "ssh ec2-user@{EC2_INSTANCE_IP} 'docker pull my-image'"
                        sh "ssh ec2-user@{EC2_INSTANCE_IP} 'docker run -d -p 80:8080 my-image'"
                    }
                }
            }
        }
    }
}
