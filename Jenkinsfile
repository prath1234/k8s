pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        ECR_REPO = '163434000537.dkr.ecr.ap-south-1.amazonaws.com/nginx-app'
        IMAGE_TAG = 'latest'
        CLUSTER_NAME = 'pro-eks-cluster'
    }

    stages {

        stage('Checkout Code') {
    steps {
        git branch: 'main', url: 'https://github.com/prath1234/k8s.git'
    }
}

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t nginx-app .'
            }
        }

        stage('Tag Image') {
            steps {
                sh '''
                docker tag nginx-app:latest $ECR_REPO:$IMAGE_TAG
                '''
            }
        }

         stage('Login to ECR') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws'
                ]]) {
                    sh '''
                    aws ecr get-login-password --region $AWS_REGION \
                    | docker login --username AWS --password-stdin $ECR_REPO
                    '''
                }
            }
        }

        stage('Push Image to ECR') {
            steps {
                sh 'docker push $ECR_REPO:$IMAGE_TAG'
            }
        }

        stage('Deploy to EKS') {
    steps {
        withCredentials([[
            $class: 'AmazonWebServicesCredentialsBinding',
            credentialsId: 'aws'
        ]]) {
            sh '''
            aws eks update-kubeconfig --region ap-south-1 --name pro-eks-cluster

            sed -i "s|IMAGE_URI|163434000537.dkr.ecr.ap-south-1.amazonaws.com/nginx-app:latest|g" deployment.yaml

            kubectl apply -f deployment.yaml
            kubectl apply -f service.yaml
            '''
        }
    }
}
}
}
