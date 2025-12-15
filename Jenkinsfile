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
    withCredentials([usernamePassword(credentialsId: 'DOCKERHUB_CREDENTIALS',
                                      usernameVariable: 'DH_USER',
                                      passwordVariable: 'DH_PASS')]) {
      sh '''
        set -e
        IMAGE="$DH_USER/$IMAGE_NAME:latest"

        # Render Spring deployment with the right Docker image
        sed "s|IMAGE_PLACEHOLDER|$IMAGE|g" k8s/spring-deployment.yaml > k8s/spring-deployment.rendered.yaml

        # Apply namespace + mysql + spring (order matters)
        docker run --rm \
          -v "/home/jenkins/.kube:/root/.kube:ro" \
          -v "$(pwd):/work" -w /work \
          bitnami/kubectl:latest \
          kubectl apply -f k8s/namespace.yaml

        docker run --rm \
          -v "/home/jenkins/.kube:/root/.kube:ro" \
          -v "$(pwd):/work" -w /work \
          bitnami/kubectl:latest \
          kubectl apply -f k8s/mysql-deployment.yaml -f k8s/mysql-service.yaml

        docker run --rm \
          -v "/home/jenkins/.kube:/root/.kube:ro" \
          -v "$(pwd):/work" -w /work \
          bitnami/kubectl:latest \
          kubectl apply -f k8s/spring-deployment.rendered.yaml -f k8s/spring-service.yaml

        docker run --rm \
          -v "/home/jenkins/.kube:/root/.kube:ro" \
          bitnami/kubectl:latest \
          kubectl get pods,svc -A
      '''
    }
  }
}


  }
}
