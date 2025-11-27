pipeline {
    agent any
    
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/prischepovna/java-maven-ci-demo.git'
            }
        }
        
        stage('Verify Project') {
            steps {
                bat '''
                echo "=== Checking Project Structure ==="
                dir
                echo "=== Checking Java ==="  
                java -version
                echo "=== Checking Files ==="
                dir src /s
                '''
            }
        }
        
        stage('Compile and Run') {
            steps {
                bat '''
                echo "=== Compiling Java ==="
                javac src/main/java/com/example/App.java -d target/classes
                echo "=== Running Application ==="
                java -cp target/classes com.example.App
                '''
            }
        }
        
        stage('Create JAR') {
            steps {
                bat '''
                echo "=== Creating JAR File ==="
                echo Main-Class: com.example.App > manifest.txt
                jar cvfm myapp.jar manifest.txt -C target/classes .
                dir *.jar
                '''
                archiveArtifacts 'myapp.jar'
            }
        }
    }
    
    post {
        always {
            echo "=== CI/CD Pipeline Complete ==="
        }
        success {
            echo "✅ SUCCESS - Full CI/CD completed!"
        }
    }
}