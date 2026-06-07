pipeline {
    agent { label 'slave' }

    environment {
        APP_NAME = 'flask-app'
        DEPLOY_SERVER = '100.26.187.5'    // app-server private IP
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
                        # 1. Create deploy directory using sudo, then grant ownership to the ubuntu user
                        ssh -o StrictHostKeyChecking=no ${DEPLOY_USER}@${DEPLOY_SERVER} \
                            "sudo mkdir -p ${DEPLOY_PATH} && sudo chown -R ${DEPLOY_USER}:${DEPLOY_USER} ${DEPLOY_PATH}"

                        # 2. Copy all project files to app-server (will succeed now that permissions are fixed)
                        scp -o StrictHostKeyChecking=no -r . ${DEPLOY_USER}@${DEPLOY_SERVER}:${DEPLOY_PATH}/

                        # 3. Run docker compose on app-server (using double quotes so Jenkins environment variables expand correctly)
                        ssh -o StrictHostKeyChecking=no ${DEPLOY_USER}@${DEPLOY_SERVER} "
                            cd ${DEPLOY_PATH}
                            sudo docker compose down || true
                            sudo docker compose up -d --build
                            echo 'Running containers:'
                            sudo docker ps
                        "
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
