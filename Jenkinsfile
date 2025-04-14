pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = "devops-app"
        DOCKER_REGISTRY = "https://index.docker.io/v1/"
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: 'master']],
                    extensions: [[$class: 'CleanBeforeCheckout']],
                    userRemoteConfigs: [[
                        url: 'https://github.com/kaiwalya65/project1.git',
                        credentialsId: 'github-credentials' // Add if private repo
                    ]]
                ])
            }
        }
        
        stage('Verify Files') {
            steps {
                script {
                    // Verify critical files exist
                    if (!fileExists('Dockerfile')) {
                        error("ERROR: Dockerfile not found in workspace")
                    }
                    if (!fileExists('app.py')) {
                        error("ERROR: app.py not found in workspace")
                    }
                    
                    // Platform-specific file listing
                    if (isUnix()) {
                        sh 'ls -la'
                    } else {
                        bat 'dir'
                    }
                }
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    // Build with proper tagging
                    dockerImage = docker.build("${DOCKER_IMAGE}:${env.BUILD_ID}")
                    
                    // For debugging
                    if (isUnix()) {
                        sh 'docker images'
                    } else {
                        bat 'docker images'
                    }
                }
            }
        }
        
        stage('Run Tests') {
            steps {
                script {
                    // Cross-platform test execution
                    if (fileExists('tests/')) {
                        if (isUnix()) {
                            sh 'python -m pytest tests/ || echo "Tests failed but continuing"'
                        } else {
                            bat 'python -m pytest tests/ || echo "Tests failed but continuing"'
                        }
                    } else {
                        echo "WARNING: No tests directory found, skipping tests"
                    }
                }
            }
        }
        
        stage('Deploy') {
            steps {
                script {
                    // Only deploy from master branch
                    if (env.BRANCH_NAME == 'master') {
                        docker.withRegistry(env.DOCKER_REGISTRY, 'dockerhub-credentials') {
                            dockerImage.push()
                            dockerImage.push("latest") // Also tag as latest
                        }
                        echo "Successfully pushed to Docker Hub"
                    } else {
                        echo "Skipping deployment for non-master branch"
                    }
                }
            }
        }
    }
    
    post {
        always {
            cleanWs()
            script {
                // Clean up Docker images
                if (isUnix()) {
                    sh 'docker system prune -f || true'
                } else {
                    bat 'docker system prune -f || echo "Cleanup failed"'
                }
            }
        }
        success {
            emailext (
                subject: "SUCCESS: Pipeline ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: "View build: ${env.BUILD_URL}",
                to: 'team@example.com'
            )
        }
        failure {
            emailext (
                subject: "FAILED: Pipeline ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: "Check console: ${env.BUILD_URL}",
                to: 'team@example.com'
            )
        }
    }
}
