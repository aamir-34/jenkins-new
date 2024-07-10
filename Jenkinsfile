pipeline {
    agent any

    environment {
        PROJECT_ID = 'high-splicer-429006-m7'
        REPOSITORY = 'us-central1-docker.pkg.dev/high-splicer-429006-m7/repo-image'
        IMAGE_NAME = 'aamir-latest'
        GCP_CREDENTIALS = credentials('gcp-service-account')
        GCP_KEY_FILE    = 'high-splicer-429006-m7-b3873a9f89ed.json'
    }

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/aamir-34/jenkins-new.git'
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${IMAGE_NAME}")
                }
            }
        }
        stage('Push to GCP Artifact Registry') {
            steps {
                script {
                    withCredentials([file(credentialsId: 'gcp-service-account', variable: 'GCP_KEY_FILE')]) {
                        sh '''
                            # Authenticate with GCP
                            gcloud auth activate-service-account --key-file=${GCP_KEY_FILE}
                            gcloud config set project ${PROJECT_ID}

                            # Configure Docker to use the gcloud command-line tool as a credential helper
                            gcloud auth configure-docker

                            # Tag and push the Docker image
                            docker tag ${IMAGE_NAME}:latest ${PROJECT_ID}-docker.pkg.dev/${PROJECT_ID}/${REPOSITORY}/${IMAGE_NAME}:latest
                            docker push ${PROJECT_ID}-docker.pkg.dev/${PROJECT_ID}/${REPOSITORY}/${IMAGE_NAME}:latest
                        '''
                    }
                }
            }
        }
    }
}
