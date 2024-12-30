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
        sh 'rm -f minikube'
        // build the project and create a JAR file
        sh 'mvn clean package -DskipTests'
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
            sh '''
                apt-get update \
                && apt-get install -y curl \
                && curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl" \
                && chmod 755 kubectl \
                && mv kubectl /bin
               '''
            withKubeConfig([namespace: "this-other-namespace"]) {
                    sh 'curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 \
                          && chmod +x minikube'
                    sh 'sudo mkdir -p /usr/local/bin/ \
                        && sudo install minikube /usr/local/bin/'
                    sh 'apt-get install conntrack -y'
                    sh 'curl --output crictl-v1.26.0-linux-amd64.tar.gz "https://github.com/kubernetes-sigs/cri-tools/releases/download/v1.26.0/crictl-v1.26.0-linux-amd64.tar.gz"'
                    sh 'sudo tar zxvf crictl-v1.26.0-linux-amd64.tar.gz -C /usr/local/bin '
                    sh 'minikube start --vm-driver=none --force'
                    sh 'kubectl apply -f k8s/db.yml'
                    sh 'kubectl apply -f k8s/petclinic.yml'
            }
        }
    }
  }
}
