pipeline {
    agent any

    environment {
        DOCKERHUB_USER = 'vikasrawatengg'
        IMAGE_NAME     = 'lms-platform'
        DOCKER_CREDS   = 'dockerhub-credentials'
        APP_SERVER_1   = '44.205.2.115'
        APP_SERVER_2   = '3.80.144.215'
    }

    stages {
        stage('1. Source Code Management') {
            steps {
                checkout scm
            }
        }

        stage('2. Build Application') {
            steps {
                sh 'ls -la && test -f index.html'
            }
        }

        stage('3. Docker Image Creation') {
            steps {
                sh "docker build -t ${DOCKERHUB_USER}/${IMAGE_NAME}:latest ."
            }
        }

        stage('4. Docker Image Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${DOCKER_CREDS}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh "echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin"
                    sh "docker push ${DOCKERHUB_USER}/${IMAGE_NAME}:latest"
                }
            }
        }

        stage('5 & 6. Application Deployment & Container Management') {
            steps {
                sshagent(credentials: ['app-servers-ssh']) {
                    sh """
                    for IP in ${APP_SERVER_1} ${APP_SERVER_2}; do
                        ssh -o StrictHostKeyChecking=no ubuntu@\$IP "
                            docker pull ${DOCKERHUB_USER}/${IMAGE_NAME}:latest &&
                            docker stop lms-container || true &&
                            docker rm lms-container || true &&
                            docker run -d --name lms-container --env-file /opt/lms-app/.env -p 8081:80 --restart=always ${DOCKERHUB_USER}/${IMAGE_NAME}:latest
                        "
                    done
                    """
                }
            }
        }

        stage('7. Deployment Verification') {
            steps {
                sh """
                sleep 5
                curl -f http://${APP_SERVER_1} | head -n 10
                curl -f http://${APP_SERVER_2} | head -n 10
                """
            }
        }
    }
}
