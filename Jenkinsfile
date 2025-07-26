pipeline {
    agent any

    environment {
        DOCKER_HUB_REPO = 'vennilavan12/trend-prod'
        IMAGE_TAG = "${env.BUILD_NUMBER}"
    }

    stages {
        stage('Clone Repo') {
            steps {
                // Clones the main branch from your GitHub repository
                git branch: 'main', url: 'https://github.com/Dharsh-creater/Trend.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Builds the Docker image and tags it with the build number
                    docker.build("${DOCKER_HUB_REPO}:${IMAGE_TAG}")
                }
            }
        }

        stage('Login to DockerHub') {
            steps {
                // Logs in to DockerHub using Jenkins credentials
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    sh 'echo $PASSWORD | docker login -u $USERNAME --password-stdin'
                }
            }
        }

        stage('Push Image') {
            steps {
                script {
                    // Pushes the image with both the build number and 'latest' tags
                    docker.image("${DOCKER_HUB_REPO}:${IMAGE_TAG}").push()
                    docker.image("${DOCKER_HUB_REPO}:${IMAGE_TAG}").push("latest")
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                // Uses the kubeconfig file credential to update the deployment in EKS
                withCredentials([file(credentialsId: 'kubeconfig-file', variable: 'KUBECONFIG')]) {
                    sh '''
                      export KUBECONFIG=$KUBECONFIG
                      kubectl set image deployment/trend-deployment trend-container=${DOCKER_HUB_REPO}:${IMAGE_TAG} --namespace=default
                    '''
                }
            }
        }

        stage('Cleanup') {
            steps {
                // Cleans up unused Docker images and logs out of DockerHub
                sh 'docker image prune -f'
                sh 'docker logout'
            }
        }
    }

    post {
        always {
            echo 'Pipeline execution completed!'
        }
        success {
            echo 'Pipeline executed successfully!'
        }
        failure {
            echo 'Pipeline failed! Check the logs for details.'
        }
    }
}   
