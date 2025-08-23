pipeline {
    agent any

    environment {
        PROJECT_ID = 'webapp-pipeline-project'
        REGION = 'us-central1'
        CLUSTER_NAME = 'my-cluster'
        IMAGE_NAME = "us-central1-docker.pkg.dev/${PROJECT_ID}/app-repo/hello-node-app"
    }

    stages {

        stage('GCP Auth') {
            steps {
                withCredentials([file(credentialsId: 'gcp-sa-key', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                    sh 'gcloud auth activate-service-account --key-file=$GOOGLE_APPLICATION_CREDENTIALS'
                    sh "gcloud config set project $PROJECT_ID"
                }
            }
        }

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${IMAGE_NAME}:latest ./app"
                }
            }
        }

        stage('Push to Artifact Registry') {
            steps {
                script {
                    sh "gcloud auth configure-docker us-central1-docker.pkg.dev --quiet"
                    sh "docker push ${IMAGE_NAME}:latest"
                }
            }
        }

        stage('Connect to GKE') {
            steps {
                script {
                    sh "gcloud container clusters get-credentials $CLUSTER_NAME --region $REGION --project $PROJECT_ID"
                }
            }
        }

        stage('Deploy to GKE') {
            steps {
                script {
                    sh "kubectl apply -f k8s/"
                }
            }
        }

    }

    post {
        success {
            echo 'Pipeline successfull!'
        }
        failure {
            echo 'Pipeline error!'
        }
    }
}
