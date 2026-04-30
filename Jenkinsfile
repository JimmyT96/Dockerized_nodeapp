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
        EKS_CLUSTER_NAME = 'your-eks-cluster-name'
        AWS_REGION = 'us-east-1'
    }

    agent any

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
                        
                        
                        sh "kubectl set image deployment/springboot springboot=${FULL_IMAGE_NAME} --record"
                        
                
                        sh "kubectl rollout status deployment/springboot"
                    }
                }
            }
        }
    }
    
    post {
        always {
            sh "docker rmi ${FULL_IMAGE_NAME} || true"
        }
    }
}
