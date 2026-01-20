pipeline {
  agent any

  tools {
    jdk 'Java17'
    maven 'Maven'
  }

  environment {
    DOCKER_USER = "wipakhilesh"
    IMAGE_NAME = "indiaproj"
    IMAGE_TAG = "1.0"
  }

  stages {

    stage('Checkout Code') {
      steps {
        git branch: 'master',
            credentialsId: 'github-key',
            url: 'https://github.com/sunnysingh94275-ai/k8test.git'
      }
    }

      post {
        always {
          junit '**/target/surefire-reports/*S.xml'
        }
      }
    }

    stage('Build JAR') {
      steps {
        bat 'mvn clean package -DskipTests'
      }
    }

    stage('Build Docker Image') {
      steps {
        bat """
			cd indiaproj docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
        """
      }
    }

    stage('Push Docker Image to DockerHub') {
      steps {
        echo 'Pushing Docker Image'
        withCredentials([string(credentialsId: 'dockerhubpwd', variable: 'DOCKER_PASS')]) {
          bat '''
            echo "$DOCKER_PASS" | docker login -u wipakhilesh --password-stdin
            docker push wipakhilesh/indiaproj:latest
          '''
        }
      }
    }

    stage('Deploy Project to K8s') {
      steps {
        bat """
        minikube delete
        minikube start
        minikube image load wipakhilesh/indiaproj:1.0
        kubectl apply -f deployment.yaml
        kubectl apply -f services.yaml
        kubectl get pods
        kubectl get services
        minikube addons enable dashboard
        """
      }
    }
  }

  post {
    success {
      echo 'Pipeline Executed Successfully!'
    }
    failure {
      echo 'Pipeline Failed — Check Logs'
    }
  }
}
