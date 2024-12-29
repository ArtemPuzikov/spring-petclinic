pipeline {
  agent any
  parameters {
    string(name: 'build_version', defaultValue: 'V1.0', description: 'Build version to use for Docker image')
  }
  stages {
    stage('Checkout') {
      steps {
        git branch: 'main', url: 'https://github.com/ArtemPuzikov/spring-petclinic.git'
      }
    }
    stage('Build and Test') {
      steps {
        sh 'ls -ltr'
        // build the project and create a JAR file
        sh 'mvn clean package'
      }
    }
    stage('Build and Push Docker Image') {
      environment {
        DOCKER_IMAGE = "artempuzikov/spring-petclinic:latest"
        REGISTRY_CREDENTIALS = credentials('dockerhub')
      }
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'password', usernameVariable: 'username')]) {
            sh '''
                docker build -t ${DOCKER_IMAGE} .
                docker login -u $username -p $password
                docker push ${DOCKER_IMAGE}
          '''
        }
      }
    }
    stage('Run compose') {
        steps {
            sh 'docker compose up -d'
            sh 'docker compose ps'
        }
    }
    stage('Deploying App to Minikube') {
        steps {
            sh 'kubectl apply -f db.yml'
            sh 'kubectl apply -f petclinic.yml'
        }
    }
  }
}
