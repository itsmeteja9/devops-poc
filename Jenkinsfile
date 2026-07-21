pipeline {
    agent any

    environment {
        PROJECT_ID = "devops-poc-demo"
        REGION = "us-central1"
        REPO = "hello-repo"
        IMAGE = "hello-world"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Google Cloud SDK') {
            steps {
                bat '''
                curl -O https://dl.google.com/dl/cloudsdk/channels/rapid/downloads/google-cloud-cli-474.0.0-windows-x86_64.zip
                tar -xf google-cloud-cli-474.0.0-windows-x86_64.zip
                google-cloud-sdk\\install.bat
                '''
            }
        }

        stage('Authenticate to GCP') {
            steps {
                withCredentials([file(credentialsId: 'gcp-sa-key', variable: 'GCP_KEY')]) {
                    bat '''
                    google-cloud-sdk\\bin\\gcloud auth activate-service-account --key-file=%GCP_KEY%
                    google-cloud-sdk\\bin\\gcloud config set project %PROJECT_ID%
                    '''
                }
            }
        }

        stage('Configure Docker for Artifact Registry') {
            steps {
                bat '''
                google-cloud-sdk\\bin\\gcloud auth configure-docker %REGION%-docker.pkg.dev --quiet
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                bat '''
                docker build -t %REGION%-docker.pkg.dev/%PROJECT_ID%/%REPO%/%IMAGE%:%BUILD_NUMBER% .
                '''
            }
        }

        stage('Push Docker Image') {
            steps {
                bat '''
                docker push %REGION%-docker.pkg.dev/%PROJECT_ID%/%REPO%/%IMAGE%:%BUILD_NUMBER%
                '''
            }
        }

        stage('Deploy to Cloud Run') {
            when {
                branch 'main'
            }
            steps {
                bat '''
                google-cloud-sdk\\bin\\gcloud run deploy %IMAGE% ^
                    --image %REGION%-docker.pkg.dev/%PROJECT_ID%/%REPO%/%IMAGE%:%BUILD_NUMBER% ^
                    --region %REGION% ^
                    --platform managed ^
                    --allow-unauthenticated
                '''
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
