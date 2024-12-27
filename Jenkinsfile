pipeline {
    agent any

    tools {
        // Use the configured JDK and Maven versions in Jenkins
        jdk 'JDK11'
        maven 'Maven3'
    }

    environment {
        // Define SonarQube environment variables
        SONARQUBE_SERVER = 'SonarQube_Server' // Name configured in Jenkins
        SONARQUBE_TOKEN = credentials('sonarqube-token') // Add the token in Jenkins Credentials
    }

    stages {
        stage('Checkout') {
            steps {
                // Checkout source code from Git
                checkout scm
            }
        }

        stage('Build') {
            steps {
                // Run Maven clean and package
                sh 'mvn clean package'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                // Run SonarQube analysis
                withSonarQubeEnv('SonarQube_Server') { // SonarQube server name in Jenkins
                    sh 'mvn sonar:sonar -Dsonar.login=$SONARQUBE_TOKEN'
                }
            }
        }

        stage('Quality Gate') {
            steps {
                // Wait for SonarQube Quality Gate
                timeout(time: 1, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
    }

    post {
        always {
            // Archive artifacts for later analysis
            archiveArtifacts artifacts: '**/target/*.jar', allowEmptyArchive: true

            // Clean up workspace
            cleanWs()
        }

        failure {
            // Notify failure (optional, e.g., via email or Slack)
            echo 'Build failed!'
        }

        success {
            // Notify success (optional)
            echo 'Build and SonarQube analysis successful!'
        }
    }
}