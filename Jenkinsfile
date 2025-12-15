pipeline {
  agent any

  environment {
    DOCKER_IMAGE = "samiwin1/students-management-devopss"   // change if needed
    DOCKERHUB_CREDENTIALS_ID = "DOCKERHUB_CREDENTIALS"     // change if your ID differs
    K8S_NAMESPACE = "default"
  }

  stages {

    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build Spring Boot (Maven)') {
      steps {
        sh '''
          set -e
          if [ -f "./mvnw" ]; then
            chmod +x mvnw || true
            ./mvnw -DskipTests clean package
          else
            mvn -DskipTests clean package
          fi
        '''
      }
    }

    // ✅ Optional: SonarQube stage (disable if you don’t have Sonar)
    stage('SonarQube Analysis (Optional)') {
      when { expression { return env.SONARQUBE_ENABLED == 'true' } }
      steps {
        echo "SonarQube enabled"
        withSonarQubeEnv('sonarqube') {
          sh '''
            if [ -f "./mvnw" ]; then
              ./mvnw sonar:sonar
            else
              mvn sonar:sonar
            fi
          '''
        }
      }
    }

    stage('Docker Build') {
      steps {
        sh '''
          set -e
          docker build -t $DOCKER_IMAGE:$BUILD_NUMBER -t $DOCKER_IMAGE:latest .
        '''
      }
    }

    stage('Docker Push') {
      steps {
        withCredentials([usernamePassword(credentialsId: "${DOCKERHUB_CREDENTIALS_ID}", usernameVariable: 'DH_USER', passwordVariable: 'DH_PASS')]) {
          sh '''
            set -e
            echo "$DH_PASS" | docker login -u "$DH_USER" --password-stdin
            docker push $DOCKER_IMAGE:$BUILD_NUMBER
            docker push $DOCKER_IMAGE:latest
            docker logout
          '''
        }
      }
    }

    // ✅ Optional: Kubernetes deploy (requires kubectl configured on Jenkins node)
    stage('Deploy to Kubernetes (Optional)') {
      when { expression { return env.K8S_ENABLED == 'true' } }
      steps {
        sh '''
          set -e
          kubectl apply -n $K8S_NAMESPACE -f k8s/mysql-deployment.yaml || true
          kubectl apply -n $K8S_NAMESPACE -f k8s/spring-deployment.yaml || true
          kubectl rollout status -n $K8S_NAMESPACE deployment/mysql-deployment || true
          kubectl rollout status -n $K8S_NAMESPACE deployment/spring-deployment || true
        '''
      }
    }
  }

  post {
    always {
      echo "Pipeline finished."
    }
  }
}
