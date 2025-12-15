pipeline {
  agent any

  environment {
    IMAGE = "samiwin1/students-management-devopss"
  }

  stages {
    stage('Checkout') {
      steps { checkout scm }
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
        sh 'docker build -t $IMAGE:latest .'
      }
    }

    stage('Docker Login & Push') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'DOCKERHUB_CREDENTIALS', usernameVariable: 'DH_USER', passwordVariable: 'DH_PASS')]) {
          sh '''
            echo "$DH_PASS" | docker login -u "$DH_USER" --password-stdin
            docker push $IMAGE:latest
            docker logout
          '''
        }
      }
    }
  }
}
