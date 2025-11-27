pipeline {
    agent any

    options {
        timestamps()
        buildDiscarder(logRotator(numToKeepStr: '20'))
        disableConcurrentBuilds()
    }

    environment {
        MAVEN_OPTS = '-Dmaven.test.failure.ignore=false'
    }

    triggers {
        pollSCM('H/2 * * * *')
    }

    stages {
        stage('Fix Git Ownership') {
            steps {
                bat 'git config --global --add safe.directory D:/repos/java-maven-ci-demo.git'
            }
        }

        stage('Checkout') {
            steps {
                checkout([$class: 'GitSCM',
                    branches: [[name: '*/${BRANCH_NAME ?: "develop"}']],
                    userRemoteConfigs: [[url: 'file:///D:/repos/java-maven-ci-demo.git']]
                ])
            }
        }

        stage('Build') {
            steps {
                bat 'mvn -v'
                bat 'mvn -B -U clean compile'
            }
        }

        stage('Test') {
            steps {
                bat 'mvn -B test'
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                    publishHTML([reportDir: 'target/site/jacoco', 
                                reportFiles: 'index.html', 
                                reportName: 'JaCoCo Coverage'])
                }
            }
        }

        stage('Package') {
            steps {
                bat 'mvn -B package'
            }
            post {
                success {
                    archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
                }
            }
        }

        stage('Quality gates') {
            when {
                anyOf {
                    branch 'develop'
                    branch 'main'
                }
            }
            steps {
                echo 'Quality checks placeholder: линтеры, статанализ, SonarQube и т.д.'
            }
        }

        stage('Deploy (local)') {
            when {
                branch 'main'
            }
            steps {
                bat 'nohup java -jar target/java-maven-ci-demo-1.0.0.jar > app.log 2>&1 &'
            }
        }
    }

    post {
        success {
            echo "Build successful for ${env.BRANCH_NAME}"
        }
        failure {
            echo "Build failed for ${env.BRANCH_NAME}"
        }
    }
}