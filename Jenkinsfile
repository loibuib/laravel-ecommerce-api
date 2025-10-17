pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/loibuib/laravel-ecommerce-api'
            }
        }

        stage('Build') {
            steps {
                // Your build steps (e.g., Maven, Gradle)
                sh 'mvn clean install'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv(installationName: 'sonarqube-server') { // Use the name configured in Jenkins
                    // Execute the SonarQube Scanner
                    sh 'sonar-scanner -Dsonar.projectKey=your_project_key -Dsonar.sources=src'
                    // Add other SonarQube properties as needed (e.g., -Dsonar.java.binaries=target/classes)
                }
            }
        }

        stage('Quality Gate Check') {
            steps {
                // Optional: Wait for SonarQube analysis to complete and check quality gate status
                waitForQualityGate abortPipeline: true
            }
        }
    }
}
