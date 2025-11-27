pipeline {
    agent any
    
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', 
                    url: 'https://github.com/prischepovna/java-maven-ci-demo.git'
            }
        }
        
        stage('Build') {
            steps {
                bat 'echo "=== Starting Maven Build ==="'
                bat 'mvn --version'
                bat 'mvn clean compile'
            }
        }
        
        stage('Test') {
            steps {
                bat 'echo "=== Running Tests ==="'
                bat 'mvn test'
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                }
            }
        }
        
        stage('Package') {
            steps {
                bat 'echo "=== Creating JAR ==="'
                bat 'mvn package'
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        }
    }
    
    post {
        always {
            echo "=== Build completed ==="
        }
    }
}