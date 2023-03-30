pipeline {
  agent any

  environment {
    DOCKERHUB_CREDENTIALS = 'dockerhubcred'
    REMOTE_SERVER = '13.212.68.40'
    REMOTE_USER = 'ubuntu'            
  }

  // Fetch code from GitHub

 stages {
        stage('Checkout') {
            steps {
                // Checkout source code from Git repository
                git 'https://github.com/Abdgs/DevOps-CI-CD-Project.git'
            }
        }
        
        stage('Build') {
            steps {
                // Run Maven to build the project
                sh 'mvn clean install'
            }
        }
        
        stage('Test') {
            steps {
                // Run tests using Maven
                sh 'mvn test'
            }
        }
        
        stage('Package') {
            steps {
                // Create a JAR file using Maven
                sh 'mvn package'
            }
            post {
                // Archive the JAR file as a build artifact
                archiveArtifacts artifacts: 'target/myproject.jar', fingerprint: true
            }
        }
    }
}
  // Test Java application

    stage('Maven Test') {
      steps {
        sh 'mvn test'
      }
    }

   // Build docker image in Jenkins

    stage('Build Docker Image') {

      steps {
        sh 'docker build -t javawebapp:latest .'
        sh 'docker tag javawebapp palakbhawsar/javawebapp:latest'
      }
    }

   // Login to DockerHub before pushing docker Image

    stage('Login to DockerHub') {
      steps {
        sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u    $DOCKERHUB_CREDENTIALS_USR --password-stdin'
      }
    }

   // Push image to DockerHub registry

    stage('Push Image to dockerHUb') {
      steps {
        sh 'docker push palakbhawsar/javawebapp:latest'
      }
      post {
        always {
          sh 'docker logout'
        }
      }

    }

   // Pull docker image from DockerHub and run in EC2 instance 

    stage('Deploy Docker image to AWS instance') {
      steps {
        script {
          sshagent(credentials: ['awscred']) {
          sh "ssh -o StrictHostKeyChecking=no 'ec2-user@54.255.155.227' docker stop javaApp || true && docker rm javaApp || true'"
      sh "ssh -o StrictHostKeyChecking=no 'ec2-user@54.255.155.227' docker pull palakbhawsar/javawebapp'"
          sh "ssh -o StrictHostKeyChecking=no ec2-user@54.255.155.227' docker run --name javaApp -d -p 8081:8081 palakbhawsar/javawebapp'"
          }
        }
      }
    }
  }
}
