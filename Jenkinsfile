pipeline {
    agent any
    
    stages {
        stage('Checkout') {
            steps {
                git branch: 'master',
                url: 'https://github.com/kaiwalya65/project1.git'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("devops-app:${env.BUILD_ID}")
                }
            }
        }
        
        stage('Run Tests') {
            steps {
                sh 'python -m pytest tests/'  
            }
        }
        
        stage('Deploy') {
            steps {
                script {
                    docker.withRegistry('', 'dockerhub-credentials') {
                        docker.image("devops-app:${env.BUILD_ID}").push()
                    }
                }
            }
        }
    }
    
    post {
        always {
            cleanWs()
        }
    }
}
