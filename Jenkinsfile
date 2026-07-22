pipeline {
    agent any

    environment {
        PROJECT_ID = "devops-poc-demo"
        REGION = "us-central1"
        REPO = "hello-repo"
        IMAGE_NAME = "devops-poc"
        IMAGE_TAG = "1"
        FULL_IMAGE_PATH = "${REGION}-docker.pkg.dev/${PROJECT_ID}/${REPO}/${IMAGE_NAME}:${IMAGE_TAG}"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Authenticate to GCP') {
            steps {
                bat """
                    gcloud auth activate-service-account --key-file=C:\\ProgramData\\Jenkins\\.jenkins\\gcp-sa-key.json
                """
            }
        }

        stage('Build Docker Image') {
            steps {
                bat """
                    docker build -t ${FULL_IMAGE_PATH} .
                """
            }
        }

        stage('Push Image to Artifact Registry') {
            steps {
                bat """
                    gcloud auth configure-docker ${REGION}-docker.pkg.dev
                    docker push ${FULL_IMAGE_PATH}
                """
            }
        }

        stage('Deploy to Cloud Run') {
            steps {
                bat """
                    gcloud run deploy ${IMAGE_NAME} ^
                        --image ${FULL_IMAGE_PATH} ^
                        --region ${REGION} ^
                        --platform managed ^
                        --allow-unauthenticated ^
                        --project ${PROJECT_ID}
                """
            }
        }
    }
}
