pipeline {
    agent { label 'slave' }

    environment {
        APP_NAME = 'flask-app'
        DEPLOY_SERVER = '10.0.1.254'    // app-server private IP
        DEPLOY_USER = 'ubuntu'
        DEPLOY_PATH = '/opt/two-tier-app'
    }

    stages {
        stage('Clone Repo') {
            steps {
                git branch: 'main', url: 'https://github.com/kbsingh10/DevOps-Project01-Two-Tier-Flask-App.git'
            }
        }

        stage('Build Image Local') {
            steps {
                echo 'Building Flask Docker image on worker...'
                sh 'docker build -t ${APP_NAME}:${BUILD_NUMBER} .'
                sh 'docker tag ${APP_NAME}:${BUILD_NUMBER} ${APP_NAME}:latest'
            }
        }

       stage('Deploy to App Server') {
            steps {
                echo 'Bundling and deploying files on app-server...'
                script {
                    withCredentials([sshUserPrivateKey(
                        credentialsId: 'jenkins-ssh-key',
                        keyFileVariable: 'SSH_KEY'
                    )]) {                   
                        sh '''
                            # 1. Compress workspace, ignoring .git, and allow exit code 1 (file changed) to pass safely
                            tar --exclude='.git' --exclude='app.tar.gz' -czf app.tar.gz . || [ $? -eq 1 ]

                            # 2. Ensure remote directory exists and is clean
                            ssh -i "$SSH_KEY" -o StrictHostKeyChecking=no "$DEPLOY_USER"@"$DEPLOY_SERVER" \
                                "sudo mkdir -p \"$DEPLOY_PATH\" && sudo chown -R $DEPLOY_USER:$DEPLOY_USER \"$DEPLOY_PATH\""

                            # 3. Copy the single archive file
                            scp -i "$SSH_KEY" -o StrictHostKeyChecking=no app.tar.gz "$DEPLOY_USER"@"$DEPLOY_SERVER":"$DEPLOY_PATH"/

                            # 4. Extract archive, clean up, and start docker compose remotely
                            ssh -i "$SSH_KEY" -o StrictHostKeyChecking=no "$DEPLOY_USER"@"$DEPLOY_SERVER" "
                                cd \"$DEPLOY_PATH\"
                                tar -xzf app.tar.gz && rm app.tar.gz
                                sudo docker compose down || true
                                sudo docker compose up -d --build
                                echo 'Running containers:'
                                sudo docker ps
                            "
                        '''
                    }
                }
            }
        }
    } // Added missing stages closing block

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
