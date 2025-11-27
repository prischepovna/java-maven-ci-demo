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
        stage('Checkout') {
            steps {
                checkout scm: [
                    $class: 'GitSCM',
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[url: 'https://github.com/prischepovna/java-maven-ci-demo.git']]
                ]
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
                echo 'Quality checks placeholder'
            }
        }

        stage('Deploy (local)') {
            when {
                branch 'main'
            }
            steps {
                bat 'java -jar target/java-maven-ci-demo-1.0.0.jar > app.log 2>&1'
            }
        }
    }

    post {
        success {
            echo "Build successful"
        }
        failure {
            echo "Build failed"
        }
    }
}