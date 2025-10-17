pipeline {
    agent any

    environment {
        SONAR_PROJECT_KEY = 'glpi'
        SONAR_PROJECT_NAME = 'glpi-test'
        SONAR_HOST_URL = 'http://10.190.200.131:9000/'
        SONAR_AUTH_TOKEN = credentials('sqa_0f384208e91ba7f6bbab7348ec907c48e915619f')
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh """
                        sonar-scanner \
                        -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                        -Dsonar.projectName=${SONAR_PROJECT_NAME} \
                        -Dsonar.sources=. \
                        -Dsonar.host.url=$SONAR_HOST_URL \
                        -Dsonar.login=$SONAR_AUTH_TOKEN
                    """
                }
            }
        }
    }
}
