pipeline {
  agent any

  environment {
    IMAGE = 'devops-poc:latest'
  }

  stages {
    stage('Checkout') {
      steps {
        git 'https://github.com/itsmeteja9/devops-poc.git'
      }
    }

    stage('Build Docker Image') {
      steps {
        sh 'docker build -t ${IMAGE} .'
      }
    }

    stage('Unit Test') {
      steps {
        sh 'npm test'
      }
    }

    stage('SonarQube Analysis') {
      environment {
        SONARQUBE = credentials('sonarqube-token')
      }
      steps {
        withSonarQubeEnv('SonarQubeServer') {
          sh 'sonar-scanner -Dsonar.projectKey=devops-poc -Dsonar.sources=.'
        }
      }
    }

    stage('Push to Local Registry') {
      steps {
        sh 'docker tag ${IMAGE} devops-poc:latest'
      }
    }

    stage('Deploy to Kubernetes') {
      steps {
        sh 'kubectl apply -f k8s/deployment.yaml'
      }
    }
  }

  post {
    success {
      echo 'Pipeline completed successfully!'
    }
    failure {
      echo 'Pipeline failed. Check logs.'
    }
  }
}
