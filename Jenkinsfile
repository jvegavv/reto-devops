pipeline {
   
     agent {
    kubernetes {
      label 'sample-app'
      defaultContainer 'jnlp'
      yaml """
apiVersion: v1
kind: Pod
metadata:
labels:
  component: ci
spec:
  # Use service account that can deploy to all namespaces
  serviceAccountName: cd-jenkins
  containers:
  - name: golang
    image: golang:1.10
    command:
    - cat
    tty: true
  - name: gcloud
    image: gcr.io/cloud-builders/gcloud
    command:
    - cat
    tty: true
  - name: kubectl
    image: gcr.io/cloud-builders/kubectl
    command:
    - cat
    tty: true
"""
}
     }

    environment {
        PROJECT_ID = 'industrial-gist-378420'
        CLUSTER_NAME = 'cluster-1'
        LOCATION = 'us-central1-c'
        CREDENTIALS_ID = 'multi-k8s'
    }
    stages {
        stage("Checkout code") {
            steps {
                checkout scm
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
        stage("Helm") {
            steps {
                    sh "helm install jenkins jenkins"   
                }
            }    
        stage('Deploy to GKE') {
            steps{
                sh "sed -i 's/hello:latest/hello:${env.BUILD_ID}/g' /jenkins/templates/deployment.yaml"
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
