pipeline {
    agent any

    tools {
        nodejs('NodeJS')
    }

    environment {
        SCANNER_HOME = tool 'SonarQubeScanner'
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NODEJS_HOME = tool name: 'NodeJS' // Assumes NodeJS is configured in Jenkins tools
        PATH = "${NODEJS_HOME}/bin:${env.PATH}"
        SONARQUBE_SERVER = 'SonarQube'  // Name configured for SonarQube in Jenkins
        NEXUS_URL = '127.0.0.1:8081' // Nexus URL
        NEXUS_REPO = 'book-repo'
        NEXUS_CREDENTIALS_ID = 'nexus-creds' // Jenkins credentials ID for Nexus
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
                withCredentials([file(credentialsId: 'npm-nexus-creds', variable: 'mynpmrc')]) {
                    echo 'Building...'
                    sh 'npm install --userconfig $mynpmrc --registry http://127.0.0.1:8081/repository/npm-book-repo --loglevel verbose'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    withSonarQubeEnv('SonarQube') {
                        sh """${SCANNER_HOME}/bin/sonar-scanner -Dsonar.host.url=http://127.0.0.1:9000/ \
                        -Dsonar.token=squ_ee990db47a9bdffa732b9fea384c2782eb8e56e1 \
                        -Dsonar.projectName="Book_API" \
                        -Dsonar.exclusions=**/node_modules/** \
                        -Dsonar.projectKey=Book_API"""
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                script {
                    sleep(time: 20, unit: 'SECONDS') // Sleep to wait for the status to update
                    def qualityGate = waitForQualityGate()
                    if (qualityGate.status == 'IN_PROGRESS') {
                        sleep(time: 20, unit: 'SECONDS') // Sleep to wait for the status to update
                        error "Quality Gate is still in progress. Retrying..."
                    }

                    if (qualityGate.status != 'OK') {
                        error "Quality Gate failed: ${qualityGate.status}"
                    }
                    else {
                        echo "Quality Gate passed: ${qualityGate.status}"
                    }
                }
            }
        }


        stage('Publish to Nexus') {
            steps {
                withCredentials([file(credentialsId: 'npm-nexus-creds', variable: 'mynpmrc')]) {
                    echo 'Publishing to Nexus...'
                    sh 'npm publish --userconfig $mynpmrc --loglevel verbose'
                }
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
