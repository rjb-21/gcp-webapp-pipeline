pipeline {
    agent any

    environment {
        PROJECT_ID = "webapp-pipeline-project"
        REGION = "us-central1"
        REPO = "app-repo"
        IMAGE = "hello-node-app"
    }

    stages {
        stage('GCP Auth') {
            steps {
                withCredentials([file(credentialsId: 'gcp-sa-key', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                sh 'gcloud auth activate-service-account --key-file=$GOOGLE_APPLICATION_CREDENTIALS'
                sh 'gcloud config set project webapp-pipeline-project'
                }
            }
        }

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/rjb-21/WebApp_Pipeline.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh 'docker build -t $REGION-docker.pkg.dev/$PROJECT_ID/$REPO/$IMAGE:$BUILD_NUMBER ./app'
                }
            }
        }

        stage('Push to Artifact Registry') {
            steps {
                script {
                    sh 'gcloud auth configure-docker $REGION-docker.pkg.dev --quiet'
                    sh 'docker push $REGION-docker.pkg.dev/$PROJECT_ID/$REPO/$IMAGE:$BUILD_NUMBER'
                }
            }
        }

        stage('Connect to GKE') {
            steps {
                sh "gcloud container clusters get-credentials my-cluster --region us-central1 --project $PROJECT_ID"
            }
        }

        stage('Deploy to GKE') {
            steps {
                script {
                    sh "kubectl apply -f k8s/"
                    sh "kubectl set image deployment/hello-node hello-node=$IMAGE_NAME:${BUILD_NUMBER}"
                }
            }
        }
    }
}
