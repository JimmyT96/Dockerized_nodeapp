/*
pipeline {

    environment {
    dockerimagename = "jemimaht/nodeapp"
    dockerImage = ""
   }

  agent any
 

  stages {

    stage('Checkout Source') {
      steps {
      git branch: 'main', url: 'https://github.com/JimmyT96/Dockerized_nodeapp'
      }
    }

    stage('Build image') {
      steps{
        script {
          dockerImage = docker.build dockerimagename
        }
      }
    }

    stage('Pushing Image') {
      environment {
               registryCredential = 'dockerhublogin'
           }
      steps{
        script {
          docker.withRegistry( 'https://registry.hub.docker.com', registryCredential ) {
            dockerImage.push("latest")
          }
        }
      }
    }

    stage("kubernetes deployment"){
      steps {
        script {
        sh 'kubectl apply -f deploymentservice.yml'
         
        }
      }
}
}

  }
*/

pipeline {
    environment {
        DOCKER_REGISTRY = "docker.io"
        DOCKER_REPO = "jemimaht/nodeapp"
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        FULL_IMAGE_NAME = "${DOCKER_REPO}:${IMAGE_TAG}"
        
        DOCKER_HUB_CREDS = 'dockerhublogin'
        AWS_CREDS_ID = 'aws-credentials-id' 
        EKS_CLUSTER_NAME = 'nodeapp-eks-prod'  
            
        AWS_REGION = 'us-east-1'
    }

    agent any

    options {
        buildDiscarder(logRotator(numToKeepStr: '10'))
        timeout(time: 30, unit: 'MINUTES')
    }

    stages {
        stage('Checkout Source') {
            steps {
                git branch: 'main', url: 'https://github.com/JimmyT96/Dockerized_nodeapp'
            }
        }

        stage('Build Image') {
            steps {
                script {
                    dockerImage = docker.build("${FULL_IMAGE_NAME}")
                }
            }
        }

        stage('Pushing Image') {
            steps {
                script {
                    docker.withRegistry("https://index.docker.io/v1/", DOCKER_HUB_CREDS) {
                        dockerImage.push("${IMAGE_TAG}")
                        dockerImage.push("latest")
                    }
                }
            }
        }

        stage('Kubernetes Deployment') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: AWS_CREDS_ID]]) {
                    script {
                        sh "aws eks update-kubeconfig --name ${EKS_CLUSTER_NAME} --region ${AWS_REGION}"
                        
                        sh "kubectl apply -f deploymentservice.yml"
                        sh "kubectl apply -f deployment.yml"
                        
                        sh "kubectl set image deployment/nodeapp-deployment nodeserver=${FULL_IMAGE_NAME} --record"
                        
                        sh "kubectl rollout status deployment/nodeapp-deployment --timeout=2m"
                    }
                }
            }
        }
    }
    
    post {
        always {
            sh "docker rmi ${FULL_IMAGE_NAME} || true"
            sh "docker rmi ${DOCKER_REPO}:latest || true"
        }
        success {
            echo "Deployed ${FULL_IMAGE_NAME} to ${EKS_CLUSTER_NAME}"
        }
    }
}
