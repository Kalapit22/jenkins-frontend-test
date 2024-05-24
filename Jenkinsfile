pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials-id')
        REPOSITORY_NAME = "frontend-equipo1"
        SERVICE_PORT = "3000" // Ajusta este puerto según el equipo
        SONARQUBE_SERVER = 'sonar-jenkins-server' // El nombre que le diste a tu servidor SonarQube
        SONARQUBE_SCANNER = 'sql1' // El nombre que le diste a tu scanner
        SONARQUBE_TOKEN = credentials('secret-sonar') // El ID que le diste a tu token
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
                        // Tag and push as 'latest'
                        sh "docker tag kalapit/${REPOSITORY_NAME}:${env.GIT_COMMIT} kalapit/${REPOSITORY_NAME}:latest"
                        sh "docker push kalapit/${REPOSITORY_NAME}:latest"
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

        stage('SonarQube Analysis') {
            environment {
                scannerHome = tool name: "${env.SONARQUBE_SCANNER}", type: 'hudson.plugins.sonar.SonarRunnerInstallation'
            }
            steps {
                withSonarQubeEnv("${env.SONARQUBE_SERVER}") {
                    sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=${env.REPOSITORY_NAME} -Dsonar.sources=./src -Dsonar.host.url=${env.SONARQUBE_SERVER} -Dsonar.login=${env.SONARQUBE_TOKEN}"
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
        always {
            script {
                def qg = waitForQualityGate()
                if (qg.status != 'OK') {
                    error "Pipeline aborted due to quality gate failure: ${qg.status}"
                }
            }
        }
    }
}
