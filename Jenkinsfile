pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials-id')
        REPOSITORY_NAME = "frontend-equipo1"
        SERVICE_PORT = "3000"
        SLACK_CHANNEL = '#your-channel'
        SONAR_SCANNER_HOME = tool 'SonarQube Scanner'
    }

    stages {
        stage('Clone repository') {
            steps {
                git url: 'https://github.com/Kalapit22/jenkins-frontend-test.git', branch: 'main'
            }
        }

        stage('Build and Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('', 'dockerhub-credentials-id') {
                        def image = docker.build("kalapit/${REPOSITORY_NAME}:${env.GIT_COMMIT}")
                        image.push()
                        sh "docker tag kalapit/${REPOSITORY_NAME}:${env.GIT_COMMIT} kalapit/${REPOSITORY_NAME}:latest"
                        sh "docker push kalapit/${REPOSITORY_NAME}:latest"
                    }
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') { // 'SonarQube' is the name of the SonarQube server configured in Jenkins
                    sh "${SONAR_SCANNER_HOME}/bin/sonar-scanner -Dsonar.projectKey=frontend-equipo1 -Dsonar.sources=."
                }
            }
        }

        stage('Update services.txt') {
            steps {
                script {
                    def serviceEntry = "${REPOSITORY_NAME}:${SERVICE_PORT}"
                    def services = readFile('/var/jenkins_home/services.txt').split('\n').collect { it.trim() }.findAll { it }
                    if (!services.contains(serviceEntry)) {
                        services << serviceEntry
                        writeFile file: '/var/jenkins_home/services.txt', text: services.join('\n')
                    }
                }
            }
        }

        stage('Deploy Services') {
            steps {
                sh 'docker-compose -f /var/jenkins_home/jenkins_docker-compose.yml pull'
                sh 'docker-compose -f /var/jenkins_home/jenkins_docker-compose.yml up -d'
            }
        }
    }

    post {
        success {
            slackSend (channel: env.SLACK_CHANNEL, color: 'good', message: "Build ${env.JOB_NAME} #${env.BUILD_NUMBER} succeeded.")
        }
        failure {
            slackSend (channel: env.SLACK_CHANNEL, color: 'danger', message: "Build ${env.JOB_NAME} #${env.BUILD_NUMBER} failed.")
        }
    }
}
