pipeline {
    agent any
    
    tools {
        jdk 'jdk23'
    }
    
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        DOCKER_TAG = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
        DOCKER_IMAGE = "jayesh9999/mental_health_assistant:${DOCKER_TAG}"
        DOCKER_BUILDKIT = '1'
        AWS_REGION = 'ap-south-1'
        CLUSTER_NAME = 'chatbot-cluster'
        KUBECONFIG = credentials('kube-config')
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', credentialsId: 'githubtoken', url: 'https://github.com/jayesh9999/mental_health_assistant.git'
            }
        }
        stage('Owasp Dependency Scan') {
            steps {
                dependencyCheck additionalArguments: '--scan ./', nvdCredentialsId: 'owaspnvdapikey', odcInstallation: 'owasp'
                sh 'ls -lR | grep dependency-check-report.xml || echo "Report file not found"'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('Sonarqube Scan') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=MentalhealthAssistant \
                    -Dsonar.projectKey=MentalhealthAssistant \
                    -Dsonar.sources=.'''
                }
            }
        }

        stage('DockerBuild') {
            steps {
                script {
                    withDockerRegistry( credentialsId: 'dockerhubcred', url: '') {
                        sh "docker pull jayesh9999/mental_health_assistant:latest || true"
                        sh "docker build --progress=plain --cache-from=${DOCKER_IMAGE} -t ${DOCKER_IMAGE} ."
                    }
                }
            }
        }

        stage('Docker Push') {
            steps {
                script {
                    withDockerRegistry( credentialsId: 'dockerhubcred', url: '') {
                        sh "docker push --quiet=false ${DOCKER_IMAGE}"
                        sh "docker rmi ${DOCKER_IMAGE} || true"
                    }
                }
            }
        }

        stage('Create IAM Role for ALB Controller') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-creds']]) {
                sh '''
                aws iam create-policy \
                --policy-name ALBIngressControllerIAMPolicy \
                --policy-document file://eks/iam/iam-policy.json || true

                eksctl create iamserviceaccount \
                --region $AWS_REGION \
                --name aws-load-balancer-controller \
                --namespace kube-system \
                --cluster $CLUSTER_NAME \
                --attach-policy-arn arn:aws:iam::621837861619:policy/ALBIngressControllerIAMPolicy \
                --approve \
                --override-existing-serviceaccounts
                '''
                }
            }
        }

        stage('Install AWS Load Balancer Controller') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-creds']]) {
                sh '''
                helm repo add eks https://aws.github.io/eks-charts || true
                helm repo update

                helm upgrade --install aws-load-balancer-controller eks/aws-load-balancer-controller \
                -n kube-system \
                --set clusterName=$CLUSTER_NAME \
                --set serviceAccount.create=false \
                --set serviceAccount.name=aws-load-balancer-controller \
                --set region=$AWS_REGION \
                --set vpcId=$(aws eks describe-cluster --region $AWS_REGION --name $CLUSTER_NAME --query "cluster.resourcesVpcConfig.vpcId" --output text) \
                --set ingressClass=alb
                '''
                }
            }
        }

        stage('Deploy App and ALB Ingress') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-creds']]) {
                withCredentials([
                    file(credentialsId: 'kube-config', variable: 'KUBECONFIG'),
                    file(credentialsId: 'k8-secret', variable: 'K8_SECRET'),
                    file(credentialsId: 'k8-app-config', variable: 'K8_CONFIG')
                    ]) {
                sh '''
                kubectl apply -f $K8_SECRET -n chatbot-app
                kubectl apply -f $K8_CONFIG -n chatbot-app
                kubectl apply -f k8s/ -n chatbot-app
                '''
                    } 
                } 
            }
        }
    }

    post {
        always {
            cleanWs()
            sh "docker image prune -f"
        }
    }
}