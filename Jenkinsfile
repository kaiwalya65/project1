pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = "devops-app"
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: 'master']],
                    userRemoteConfigs: [[url: 'https://github.com/kaiwalya65/project1.git']]
                ])
            }
        }
        
        stage('Verify Files') {
            steps {
                script {
                    // Verify critical files exist
                    if (!fileExists('Dockerfile')) {
                        error("Dockerfile not found!")
                    }
                    if (!fileExists('app.py')) {
                        error("app.py not found!")
                    }
                    sh 'ls -la'  // List all files (Unix)
                    // For Windows: bat 'dir'
                }
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    // Build with tag including build number
                    dockerImage = docker.build("${DOCKER_IMAGE}:${env.BUILD_ID}")
                }
            }
        }
        
        stage('Run Tests') {
            steps {
                script {
                    // Only run if tests directory exists
                    if (fileExists('tests/')) {
                        sh 'python -m pytest tests/ --verbose || true'
                    } else {
                        echo "No tests found, skipping test stage"
                    }
                }
            }
        }
        
        stage('Deploy') {
            steps {
                script {
                    // Only deploy if not in development branch
                    if (env.BRANCH_NAME != 'development') {
                        docker.withRegistry('https://index.docker.io/v1/', 'dockerhub-credentials') {
                            dockerImage.push()
                        }
                    }
                }
            }
        }
    }
    
    post {
        always {
            cleanWs()
        }
        failure {
            emailext (
                subject: "FAILED: Job '${env.JOB_NAME}' (${env.BUILD_NUMBER})",
                body: "Check console output at ${env.BUILD_URL}",
                to: 'team@example.com'
            )
        }
    }
}
