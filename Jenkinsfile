pipeline {
    agent any
     
    environment {
        PROJECT_ID = "devops-poc-demo"
        REGION = "us-central1"
        SERVICE = "devops-poc"
        IMAGE = "us-central1-docker.pkg.dev/devops-poc-demo/hello-repo/devops-poc"
    }

    stages {

        stage('Authenticate to GCP') {
            steps {
                withCredentials([file(credentialsId: 'gcp-sa-key', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                    bat """
gcloud auth activate-service-account --key-file=%GOOGLE_APPLICATION_CREDENTIALS%
gcloud config set project devops-poc-demo
"""
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                bat """
docker build -t us-central1-docker.pkg.dev/devops-poc-demo/hello-repo/devops-poc:%BUILD_NUMBER% .
"""
            }
        }

        stage('Push Image to Artifact Registry') {
            steps {
                bat """
gcloud auth configure-docker us-central1-docker.pkg.dev
docker push us-central1-docker.pkg.dev/devops-poc-demo/hello-repo/devops-poc:%BUILD_NUMBER%
"""
            }
        }

        stage('Deploy to Cloud Run') {
            steps {
                bat """
gcloud run deploy devops-poc ^
    --image us-central1-docker.pkg.dev/devops-poc-demo/hello-repo/devops-poc:%BUILD_NUMBER% ^
    --region us-central1 ^
    --platform managed ^
    --allow-unauthenticated ^
    --project devops-poc-demo
"""
            }
        }
    }
}
