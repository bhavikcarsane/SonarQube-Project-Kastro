pipeline {
    agent any

    tools {
        maven 'maven3'
        jdk 'jdk17'
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        DOCKERHUB_USERNAME = 'bhavikcarsane'
        DOCKER_IMAGE = "${DOCKERHUB_USERNAME}/spotify-app:latest"
    }

    stages {

        stage('Git Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Compile') {
            steps {
                sh 'mvn clean compile'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Sonar Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh """
                    ${SCANNER_HOME}/bin/sonar-scanner \
                    -Dsonar.projectName=Kastro \
                    -Dsonar.projectKey=KastroKey \
                    -Dsonar.java.binaries=target
                    """
                }
            }
        }

        stage('Build') {
            steps {
                sh 'mvn package -DskipTests'
            }
        }

        stage('Docker Build') {
            steps {
                sh 'ls -l'               // ✅ Debug: ensure Dockerfile exists
                sh 'docker version'      // ✅ Ensure docker is available
                sh "docker build -t ${DOCKER_IMAGE} ."
            }
        }

        stage('Docker Push to DockerHub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DH_USER',
                        passwordVariable: 'DH_PASS'
                    )
                ]) {
                    sh """
                    echo "${DH_PASS}" | docker login -u "${DH_USER}" --password-stdin
                    docker push ${DOCKER_IMAGE}
                    """
                }
            }
        }

        stage('Run Docker Container') {
            steps {
                sh """
                docker stop spotify-app || true
                docker rm spotify-app || true
                docker run -d --name spotify-app -p 5555:5555 ${DOCKER_IMAGE}
                """
            }
        }
    }

    post {
        always {
            echo 'Cleaning up workspace...'
            cleanWs()
        }
    }
}
