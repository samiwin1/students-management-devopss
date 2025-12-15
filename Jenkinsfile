pipeline {
  agent any

  environment {
    IMAGE_NAME = "students-management-devopss"
  }

  stages {

    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build JAR') {
      steps {
        sh '''
          chmod +x mvnw || true
          if [ -f "./mvnw" ]; then
            ./mvnw -DskipTests clean package
          else
            mvn -DskipTests clean package
          fi
        '''
      }
    }

    stage('Docker Build') {
      steps {
        withCredentials([
          usernamePassword(
            credentialsId: 'DOCKERHUB_CREDENTIALS',
            usernameVariable: 'DH_USER',
            passwordVariable: 'DH_PASS'
          )
        ]) {
          sh '''
            IMAGE="$DH_USER/$IMAGE_NAME"
            docker build -t "$IMAGE:latest" .
          '''
        }
      }
    }

    stage('Docker Login & Push') {
      steps {
        withCredentials([
          usernamePassword(
            credentialsId: 'DOCKERHUB_CREDENTIALS',
            usernameVariable: 'DH_USER',
            passwordVariable: 'DH_PASS'
          )
        ]) {
          sh '''
            IMAGE="$DH_USER/$IMAGE_NAME"
            echo "$DH_PASS" | docker login -u "$DH_USER" --password-stdin
            docker push "$IMAGE:latest"
            docker logout
          '''
        }
      }
    }
stage('Deploy to Kubernetes (Minikube)') {
  steps {
    sh '''
      set -e

      kubectl apply -f k8s/namespace.yaml
      kubectl apply -f k8s/mysql-deployment.yaml
      kubectl apply -f k8s/mysql-service.yaml
      kubectl apply -f k8s/spring-deployment.yaml
      kubectl apply -f k8s/spring-service.yaml

      kubectl get pods -A
      kubectl get svc -A
    '''
  }
}




  }
}
