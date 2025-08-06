pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = 'simple-node-app'
        DOCKER_TAG = "${BUILD_NUMBER}"
        CONTAINER_NAME = 'simple-node-app-container'
    }
    
    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out source code...'
                checkout scm
            }
        }
        
        stage('Build') {
            steps {
                echo 'Building the application...'
                sh 'npm install'
                sh 'echo "Build completed successfully"'
            }
        }
        
        stage('Test') {
            steps {
                echo 'Running tests...'
                sh 'npm test'
                sh 'echo "All tests passed"'
            }
        }
        
        stage('Docker Build') {
            steps {
                echo 'Building Docker image...'
                script {
                    sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."
                    sh "docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest"
                }
            }
        }
        
        stage('Deploy') {
            steps {
                echo 'Deploying application...'
                script {
                    // Stop and remove existing container
                    sh '''
                        docker stop ${CONTAINER_NAME} || true
                        docker rm ${CONTAINER_NAME} || true
                    '''
                    
                    // Run new container
                    sh '''
                        docker run -d \
                        --name ${CONTAINER_NAME} \
                        -p 3000:3000 \
                        ${DOCKER_IMAGE}:latest
                    '''
                }
                
                echo 'Application deployed successfully!'
            }
        }
        
        stage('Verify Deployment') {
            steps {
                echo 'Verifying deployment...'
                script {
                    sleep(time: 10, unit: 'SECONDS')
                    sh '''
                        if curl -f http://localhost:3000/health; then
                            echo "Deployment verification successful"
                        else
                            echo "Deployment verification failed"
                            exit 1
                        fi
                    '''
                }
            }
        }
    }
    
    post {
        always {
            echo 'Pipeline execution completed'
            sh 'docker system prune -f'
        }
        success {
            echo 'Pipeline executed successfully!'
        }
        failure {
            echo 'Pipeline execution failed'
            sh 'docker logs ${CONTAINER_NAME} || true'
        }
    }
}
