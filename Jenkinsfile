pipeline {
    agent any
    
    environment {
        PROJECT_ID = 'high-splicer-429006-m7'
        IMAGE_NAME = 'aamir-image'
        IMAGE_TAG = 'latest'
        REGION = 'us-central1' 
        REPOSITORY = 'us-central1-docker.pkg.dev/high-splicer-429006-m7/repo-image' 
    }
    
    stages {
        stage('Checkout') {
            steps {
                script {
                    git branch: 'main', url: 'https://github.com/aamir-34/jenkins-new.git'
                }
            }
        }
        
        stage('Build') {
            steps {
                script {
                    // Build the Docker image
                    sh "docker build -t ${REGION}-docker.pkg.dev/${PROJECT_ID}/${REPOSITORY}/${IMAGE_NAME}:${IMAGE_TAG} ."
                }
            }
        }
        
        stage('Authenticate') {
            steps {
                script {
                    // Authenticate with GCP using service account key
                    withCredentials([file(credentialsId: 'gcp-service-account-key', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                        sh "gcloud auth activate-service-account --key-file=${GOOGLE_APPLICATION_CREDENTIALS}"
                        sh "gcloud auth configure-docker ${REGION}-docker.pkg.dev"
                    }
                }
            }
        }
        
        stage('Push') {
            steps {
                script {
                    // Push the Docker image to GCP Artifact Registry
                    sh "docker push ${REGION}-docker.pkg.dev/${PROJECT_ID}/${REPOSITORY}/${IMAGE_NAME}:${IMAGE_TAG}"
                }
            }
        }
    }
    
    post {
        success {
            echo 'Pipeline succeeded! Docker image built and pushed to GCP Artifact Registry.'
        }
        failure {
            echo 'Pipeline failed! Check the Jenkins logs for details.'
        }
    }
}
