pipeline {
    agent any
    
    environment {
        PROJECT_ID = 'high-splicer-429006-m7'
        IMAGE_NAME = 'aamir-image'
        CLUSTER_NAME = 'jenkins-testing-cluster'
        IMAGE_TAG = 'latest'
        REGION = 'us-central1' 
        REPOSITORY = 'repo-image' 
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
        
        stage('Authenticate and Push') {
            steps {
                script {
                    // Authenticate with GCP using service account key
                    withCredentials([file(credentialsId: 'gcp-service-account', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                        sh "gcloud auth activate-service-account --key-file=${GOOGLE_APPLICATION_CREDENTIALS}"
                        sh "gcloud auth configure-docker ${REGION}-docker.pkg.dev"
                        
                        // Push the Docker image to GCP Artifact Registry
                        sh "docker push ${REGION}-docker.pkg.dev/${PROJECT_ID}/${REPOSITORY}/${IMAGE_NAME}:${IMAGE_TAG}"
                    }
                }
            }
        }
        
        stage('Deploy to GKE') {
            steps {
                script {
                    // Authenticate with GKE cluster
                    withCredentials([file(credentialsId: 'gcp-service-account', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                        sh "gcloud auth activate-service-account --key-file=${GOOGLE_APPLICATION_CREDENTIALS}"
                        sh "gcloud container clusters get-credentials ${CLUSTER_NAME} --region ${REGION} --project ${PROJECT_ID}"
                        
                        // Deploy the application to GKE
                        sh "kubectl set image deployment/${IMAGE_NAME} ${IMAGE_NAME}=${REGION}-docker.pkg.dev/${PROJECT_ID}/${REPOSITORY}/${IMAGE_NAME}:${IMAGE_TAG} --record"
                    }
                }
            }
        }
    }
    
    post {
        success {
            echo 'Deployment succeeded!'
        }
        failure {
            echo 'Deployment failed! Check Jenkins logs for details.'
        }
    }
}
