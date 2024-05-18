pipeline {
    agent any

    environment {
        DOCKER_CREDENTIALS_ID = 'dockerhub-credentials'
        SONARQUBE_SERVER = 'sonarqube-server'
        DATADOG_API_KEY = 'datadog-api-key'
    }

    stages {
        stage('Build') {
            steps {
                script {
                    docker.build('myapp:latest').inside {
                        sh 'npm install'
                        sh 'npm run build'
                    }
                }
                script {
                    docker.withRegistry('https://index.docker.io/v1/', DOCKER_CREDENTIALS_ID) {
                        docker.image('myapp:latest').push('latest')
                    }
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    sh 'npm run test'
                }
                junit '**/test-results.xml'
            }
        }

        stage('Code Quality Analysis') {
            steps {
                script {
                    withSonarQubeEnv('SonarQube') {
                        sh 'sonar-scanner'
                    }
                }
            }
        }

        stage('Deploy to Test') {
            steps {
                script {
                    docker.compose.up()
                }
            }
        }

        stage('Release') {
            steps {
                script {
                    octopusDeployRelease additionalArgs: '--deployTo=Production',
                        releaseVersion: '1.0.0',
                        spaceId: 'Spaces-1',
                        project: 'SIT753_Task_6.2'
                }
            }
        }

        stage('Monitoring and Alerting') {
            steps {
                script {
                    datadog(apiKey: env.DATADOG_API_KEY,
                            appKey: env.DATADOG_APP_KEY) {
                        sh 'datadog-agent status'
                    }
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
