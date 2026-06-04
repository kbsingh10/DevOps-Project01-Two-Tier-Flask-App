pipeline {
    agent { label 'slave' }

    environment {
        APP_NAME = 'flask-app'
        DEPLOY_SERVER = '10.0.1.56'    // app-server private IP
        DEPLOY_USER = 'ubuntu'
        DEPLOY_PATH = '/opt/two-tier-app'
    }

    stages {

        stage('Clone Repo') {
            steps {
                git branch: 'main', url: 'https://github.com/kbsingh10/DevOps-Project01-Two-Tier-Flask-App.git'
            }
        }

        stage('Build Image') {
            steps {
                echo 'Building Flask Docker image on worker...'
                sh 'docker build -t ${APP_NAME}:${BUILD_NUMBER} .'
                sh 'docker tag ${APP_NAME}:${BUILD_NUMBER} ${APP_NAME}:latest'
            }
        }

        stage('Deploy to App Server') {
            steps {
                echo 'Copying files and deploying on app-server...'
                sshagent(['app-server-key']) {
                    sh """
                        # Create deploy directory
                        ssh -o StrictHostKeyChecking=no ${DEPLOY_USER}@${DEPLOY_SERVER} \
                            'mkdir -p ${DEPLOY_PATH}'

                        # Copy all project files to app-server
                        scp -o StrictHostKeyChecking=no -r \
                            . \
                            ${DEPLOY_USER}@${DEPLOY_SERVER}:${DEPLOY_PATH}/

                        # Run docker compose on app-server
                        ssh -o StrictHostKeyChecking=no ${DEPLOY_USER}@${DEPLOY_SERVER} '
                            cd ${DEPLOY_PATH}
                            docker compose down || true
                            docker compose up -d --build
                            echo "Running containers:"
                            docker ps
                        '
                    """
                }
            }
        }

    }

    post {
        always {
            sh 'docker rmi ${APP_NAME}:${BUILD_NUMBER} || true'
            cleanWs()
        }
        success {
            echo "App deployed! Access at: http://${DEPLOY_SERVER}:5000"
        }
        failure {
            echo 'Pipeline failed. Check logs above.'
        }
    }
}
