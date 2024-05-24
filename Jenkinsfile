pipeline {
    agent {
        docker {
            image 'docker:stable'
            args '-v /var/run/docker.sock:/var/run/docker.sock'
        }
    }

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials-id')
        REPOSITORY_NAME = "frontend-equipo1"
        SERVICE_PORT = "3000" // Ajusta este puerto según el equipo
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
                    }
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
}
