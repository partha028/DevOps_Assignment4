pipeline {
    agent any
    environment {
        NODEJS_HOME = tool name: 'NodeJS'
        PATH = "${NODEJS_HOME}/bin:${env.PATH}"
        SONARQUBE_SERVER = 'SonarQube'
        NEXUS_URL = 'http://localhost:8081'
        NEXUS_REPO = 'your-nexus-repository-id'
        NEXUS_CREDENTIALS_ID = 'nexus-credentials-id'
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }
        stage('Build') {
            steps {
                sh 'npm run build'
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh "${SONARQUBE_SCANNER_HOME}/bin/sonar-scanner -Dsonar.projectKey=my-nodejs-project"
                }
            }
        }
        stage('Quality Gate') {
            steps {
                script {
                    timeout(time: 1, unit: 'MINUTES') {
                        waitForQualityGate abortPipeline: true
                    }
                }
            }
        }
        stage('Archive Artifact') {
            steps {
                archiveArtifacts artifacts: 'dist/**', allowEmptyArchive: true
            }
        }
        stage('Deploy to Nexus') {
            steps {
                nexusArtifactUploader artifacts: [
                    [artifactId: 'my-app', classifier: '', file: 'dist/my-app.zip', type: 'zip']
                ],
                credentialsId: "${NEXUS_CREDENTIALS_ID}",
                groupId: 'com.example',
                nexusUrl: "${NEXUS_URL}",
                repository: "${NEXUS_REPO}",
                version: '1.0.0',
                nexusVersion: 'nexus3',
                protocol: 'http'
            }
        }
    }
    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}
