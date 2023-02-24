pipeline {
    agent any
    environment {
        PROJECT_ID = 'industrial-gist-378420'
        CLUSTER_NAME = 'cluster-1'
        LOCATION = 'us-central1-c'
        CREDENTIALS_ID = 'multi-k8s'
        NODE_ENV_PATH = './venv'
        NODE_VERSION = '12.22.12'
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
                step([$class: 'KubernetesEngineBuilder'
                , projectId: env.PROJECT_ID
                , clusterName: env.CLUSTER_NAME
                , location: env.LOCATION
                , manifestPattern: 'deployment.yaml'
                , credentialsId: env.CREDENTIALS_ID
                , verifyDeployments: true])
            }
        }
    }    
}
