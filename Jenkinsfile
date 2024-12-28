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
        DOCKER_IMAGE = "latest"
        REGISTRY_CREDENTIALS = credentials('dockerhub')
      }
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'password', usernameVariable: 'username')]) {
            sh '''
                cd demo-java-app && docker build -t ${DOCKER_IMAGE} .
                docker login -u $username -p $password
                docker push artempuzikov/spring-petclinic:${DOCKER_IMAGE}
          '''
        }
      }
    }
    stage('Deploying App to Kubernetes') {
        steps {
            script {
                kubernetesDeploy(configs: "service.yml", kubeconfigId: 'kubernetes')
            }
        }
    }
  }
}
