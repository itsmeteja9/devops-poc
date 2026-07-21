pipeline {
    agent any

    environment {
        PROJECT_ID = "devops-poc-demo"
        REGION = "us-central1"
        SERVICE = "devops-poc"
        IMAGE = "us-central1-docker.pkg.dev/${PROJECT_ID}/hello-repo/devops-poc"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Authenticate to GCP') {
            steps {
                withCredentials([file(credentialsId: 'gcp-sa', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                    bat """
                        gcloud auth activate-service-account --key-file=%GOOGLE_APPLICATION_CREDENTIALS%
                        gcloud config set project ${PROJECT_ID}
                    """
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                bat """
                    docker build -t ${IMAGE}:${BUILD_NUMBER} .
                """
            }
        }

        stage('Push Image to Artifact Registry') {
            steps {
                bat """
                    gcloud auth configure-docker ${REGION}-docker.pkg.dev
                    docker push ${IMAGE}:${BUILD_NUMBER}
                """
            }
        }

        stage('Deploy to Cloud Run') {
            steps {
                bat """
                    gcloud run deploy ${SERVICE} ^
                        --image ${IMAGE}:${BUILD_NUMBER} ^
                        --region ${REGION} ^
                        --platform managed ^
                        --allow-unauthenticated
                """
            }
        }
    }
}
