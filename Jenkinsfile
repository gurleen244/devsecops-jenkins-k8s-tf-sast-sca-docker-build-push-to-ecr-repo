pipeline {
    agent any
    tools {
        maven 'Maven_3_5_2'
    }
    environment {
        AWS_REGION = 'us-east-1'
        IMAGE_NAME = 'for_our_project'
        ECR_REPO = '982081064548.dkr.ecr.us-west-2.amazonaws.com/for_our_project'
	AWS_REGION = 'us-west-2'
    }
    stages {
        stage('Compile and Run Sonar Analysis') {
            steps {
                withCredentials([string(credentialsId: 'sonarqubetokennew', variable: 'SONAR_TOKEN')]) {
                    sh '''
                        mvn clean verify sonar:sonar \
                        -Dsonar.projectKey=gurleen_gurleen \
                        -Dsonar.organization=gurleen \
                        -Dsonar.host.url=https://sonarcloud.io \
                        -Dsonar.token=$SONAR_TOKEN
                    '''
                }
            }
        }

        stage('Docker Build') {
            steps {
                script {
                    sh 'docker build -t $IMAGE_NAME .'
                }
            }
        }

        stage('Push to ECR') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-cred-id'
                ]]) {
                    script {
                        sh '''
                            # Authenticate Docker to ECR
                            aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_REPO

                            # Tag the image
                            docker tag $IMAGE_NAME:latest $ECR_REPO:latest

                            # Push the image
                            docker push $ECR_REPO:latest
                        '''
                    }
                }
            }
        }
    }
}
