pipeline {
    agent any
    environment {
        NODEJS_HOME = tool name: 'NodeJS'                      // Configured NodeJS tool in Jenkins
        PATH = "${NODEJS_HOME}/bin:${env.PATH}"
        SONARQUBE_SERVER = 'SonarQube'                         // Configured SonarQube server name in Jenkins
        SONARQUBE_SCANNER_HOME = tool name: 'SonarQubeScanner' // Configured SonarQube scanner tool in Jenkins
        NEXUS_URL = 'http://localhost:8081'                    // Nexus server URL
        NEXUS_REPO = 'nodejs-artifacts'                        // Nexus repository ID you created
        NEXUS_CREDENTIALS_ID = 'squ_723830da8e4e1602a01fb86fbe42b8d5e638a4be'          // Jenkins credentials ID for Nexus
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
                    sh 'sonar-scanner -Dsonar.projectKey=my-nodejs-project -X'
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
