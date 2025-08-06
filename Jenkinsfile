pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = 'simple-node-app'
        DOCKER_TAG = "${BUILD_NUMBER}"
        CONTAINER_NAME = 'simple-node-app-container'
        PORT = '3000'
    }
    
    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out source code from Git...'
                checkout scm
                sh 'ls -la'  // List files to verify checkout
            }
        }
        
        stage('Environment Info') {
            steps {
                echo 'Displaying environment information...'
                sh '''
                    echo "Node.js version:"
                    node --version || echo "Node.js not found"
                    echo "npm version:"
                    npm --version || echo "npm not found"
                    echo "Docker version:"
                    docker --version || echo "Docker not found"
                    echo "Current directory:"
                    pwd
                    echo "Files in directory:"
                    ls -la
                '''
            }
        }
        
        stage('Install Dependencies') {
            steps {
                echo 'Installing Node.js dependencies...'
                sh 'npm install'
                sh 'echo "Dependencies installed successfully"'
            }
        }
        
        stage('Run Tests') {
            steps {
                echo 'Running application tests...'
                sh 'npm test'
                echo 'All tests completed successfully!'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image...'
                script {
                    sh """
                        docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} .
                        docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest
                        echo "Docker image built: ${DOCKER_IMAGE}:${DOCKER_TAG}"
                    """
                }
            }
        }
        
        stage('Stop Previous Container') {
            steps {
                echo 'Stopping and removing previous container...'
                script {
                    sh '''
                        docker stop ${CONTAINER_NAME} || echo "No container to stop"
                        docker rm ${CONTAINER_NAME} || echo "No container to remove"
                        echo "Cleanup completed"
                    '''
                }
            }
        }
        
        stage('Deploy Application') {
            steps {
                echo 'Deploying new version...'
                script {
                    sh """
                        docker run -d \\
                        --name ${CONTAINER_NAME} \\
                        -p ${PORT}:3000 \\
                        --restart unless-stopped \\
                        ${DOCKER_IMAGE}:latest
                        
                        echo "Application deployed on port ${PORT}"
                    """
                }
            }
        }
        
        stage('Verify Deployment') {
            steps {
                echo 'Verifying deployment...'
                script {
                    sh '''
                        echo "Waiting for application to start..."
                        sleep 15
                        
                        echo "Testing application endpoints..."
                        
                        # Test main endpoint
                        if curl -f -s http://localhost:3000/; then
                            echo "✅ Main endpoint is working"
                        else
                            echo "❌ Main endpoint failed"
                            exit 1
                        fi
                        
                        # Test health endpoint
                        if curl -f -s http://localhost:3000/health; then
                            echo "✅ Health endpoint is working"
                        else
                            echo "❌ Health endpoint failed"
                            exit 1
                        fi
                        
                        echo "🎉 Deployment verification successful!"
                    '''
                }
            }
        }
    }
    
    post {
        always {
            echo 'Pipeline execution completed'
            sh '''
                echo "Cleaning up unused Docker images..."
                docker image prune -f || true
                echo "Current running containers:"
                docker ps
            '''
        }
        success {
            echo '🎉 Pipeline executed successfully!'
            echo "Application is running at: http://your-server-ip:${PORT}"
        }
        failure {
            echo '❌ Pipeline execution failed'
            sh '''
                echo "Checking container logs for debugging..."
                docker logs ${CONTAINER_NAME} || echo "No logs available"
                echo "Checking Docker images:"
                docker images
            '''
        }
    }
}
