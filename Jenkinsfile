pipeline {
    agent any
    
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/prischepovna/java-maven-ci-demo.git'
            }
        }
        
        stage('Build and Test') {
            steps {
                sh 'mvn --version'
                sh 'mvn clean compile test -B'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }
        
        stage('Package') {
            steps {
                sh 'mvn package -B -DskipTests'
                archiveArtifacts 'target/*.jar'
            }
        }
    }
    
    post {
        always {
            echo "CI/CD Complete!"
        }
    }
}