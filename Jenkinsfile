pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1'
        ECR_REPO   = '824445063740.dkr.ecr.us-east-1.amazonaws.com/sample-app'
        IMAGE_TAG  = "${BUILD_NUMBER}"
        CLUSTER_NAME = 'cicd-eks-cluster'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                echo "CI Trigger Successful"
            }
        }
    }
}