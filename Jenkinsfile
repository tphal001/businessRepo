pipeline {
    agent any

    environment {
        AWS_REGION   = 'us-east-1'
        ECR_REPO     = '824445063740.dkr.ecr.us-east-1.amazonaws.com/sample-app'
        IMAGE_TAG    = "${BUILD_NUMBER}"
        CLUSTER_NAME = 'cicd-eks-cluster'
        NAMESPACE    = 'app'
        DEPLOYMENT   = 'sample-app'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/tphal001/businessRepo.git'
            }
        }

        stage('Login to ECR') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-credentials'
                ]]) {
                    sh '''
                    aws ecr get-login-password --region $AWS_REGION \
                    | docker login --username AWS --password-stdin $ECR_REPO
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t $ECR_REPO:$IMAGE_TAG ."
            }
        }

        stage('Push to ECR') {
            steps {
                sh "docker push $ECR_REPO:$IMAGE_TAG"
            }
        }

        stage('Configure kubectl') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-credentials'
                ]]) {
                    sh '''
                    aws eks update-kubeconfig --name $CLUSTER_NAME --region $AWS_REGION
                    kubectl get nodes
                    '''
                }
            }
        }

        stage('Ensure Namespace & Deployment') {
            steps {
                sh '''
                # Create namespace if not exists
                kubectl get namespace $NAMESPACE || kubectl create namespace $NAMESPACE

                # Create deployment if not exists (first-time)
                kubectl get deployment $DEPLOYMENT -n $NAMESPACE || \
                kubectl create deployment $DEPLOYMENT -n $NAMESPACE --image=$ECR_REPO:$IMAGE_TAG
                '''
            }
        }
		
		stage('Create LoadBalancer (one-time)') {
			steps {
				sh '''
				# Apply LoadBalancer service (only needed once)
				kubectl apply -f service.yaml -n $NAMESPACE
				'''
			}
		}

        stage('Deploy to EKS') {
            steps {
                sh '''
                # Update deployment image
                kubectl set image deployment/$DEPLOYMENT $DEPLOYMENT=$ECR_REPO:$IMAGE_TAG -n $NAMESPACE
                kubectl rollout status deployment/$DEPLOYMENT -n $NAMESPACE
                '''
            }
        }
		
		stage('Get LoadBalancer URL') {
			steps {
				sh '''
				LB_DNS=$(kubectl get svc $DEPLOYMENT -n $NAMESPACE -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')
				echo "LoadBalancer URL: http://$LB_DNS"
				'''
			}
		}
    }

    post {
        success {
            echo "CI/CD completed successfully!"
        }
        failure {
            echo "Pipeline failed!"
        }
    }
}