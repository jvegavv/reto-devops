pipeline {
    agent any
    environment {
        PROJECT_ID = 'industrial-gist-378420'
        CLUSTER_NAME = 'cluster-1'
        LOCATION = 'us-central1-c'
        CREDENTIALS_ID = 'multi-k8s'
        NODE_ENV_PATH = './venv'
        NODE_VERSION = '12.22.12'
        GOOGLE_PROJECT_NAME = "My First Project"
        GOOGLE_APPLICATION_CREDENTIALS = credentials('multi-k8s-2')
    }
    stages {
        stage("Checkout code") {
            steps {
                checkout scm
            }
        }
        stage('Pre-cleanup') {
            steps {
                sh 'rm -rf ./venv'
                sh 'rm -rf ./node_modules'
            }
            }
            stage('Make venv') {
            steps {
                sh 'nodeenv --prebuilt -n $NODE_VERSION $NODE_ENV_PATH'
            }
            }
            stage('Install dependencies') {
            steps {
                sh '. ./venv/bin/activate && npm install'
            }
            }
            stage('Run tests') {
            steps {
                sh 'npm test'
            }
            }
        stage("Build image") {
            steps {
                script {
                    myapp = docker.build("jvegavv/hello:${env.BUILD_ID}")
                }
            }
        }
        stage("Push image") {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerID') {                        
                            myapp.push("${env.BUILD_ID}")
                    }
                }
            }
        }      
        stage('Deploy to GKE') {
            steps{
                sh "sed -i 's/hello:latest/hello:${env.BUILD_ID}/g' deployment.yaml"
                sh("gcloud auth activate-service-account --key-file=${GOOGLE_APPLICATION_CREDENTIALS}")
                sh("gcloud config set project ${PROJECT_ID}")
                sh("gcloud container clusters get-credentials cluster-1 --zone us-central1-c --project ${PROJECT_ID}")
                sh "kubectl apply -f deployment.yaml"
                sh "helm version"

            }
        }
    }    
}
