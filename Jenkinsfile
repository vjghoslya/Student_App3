pipeline {
    agent any

    environment {
        VENV_DIR = 'venv'
        DOCKER_IMAGE = 'studentapp-image:latest'
        DOCKER_CONTAINER = 'studentapp-container'
        STAGING_SERVER = '192.168.188.142'
        REMOTE_USER = 'student'
    }

    stages {
        stage('Clone Repo') {
            steps {
                git branch: 'main', url: 'https://github.com/vjghoslya/studentapp3.git'
            }
        }

        stage('Install Dependencies in Virtualenv') {
            steps {
                sh '''
                    python3 -m venv $VENV_DIR
                    . $VENV_DIR/bin/activate
                    pip install --upgrade pip
                    pip install -r requirements.txt
                '''
            }
        }

        stage('Run Tests') {
            steps {
                sh '''
                    . $VENV_DIR/bin/activate
                    pytest test_app.py
                '''
            }
        }

        stage('Dockerize Application') {
            when {
                expression { currentBuild.result == null || currentBuild.result == 'SUCCESS' }
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKERHUB_USER', passwordVariable: 'DOCKERHUB_PASS')]) {
                    sh '''
                        docker build -t studentapp-image:latest .
                        docker tag studentapp-image:latest vjghoslya/studentapp-image:latest
                        echo "$DOCKERHUB_PASS" | docker login -u "$DOCKERHUB_USER" --password-stdin
                        docker push vjghoslya/studentapp-image:latest
                    '''
                }
            }
        }

        stage('SSH and run multiple commands') {
            steps {
                script {
                    sshagent (credentials: ['student-node']) {
                        sh """
                            ssh -o StrictHostKeyChecking=no ${env.$REMOTE_USER}@${env.$STAGING_SERVER} << EOF
                                echo "Connected to \$(hostname)"
                                pwd
                                ls -l /var/log
                                uptime
                                docker pull $DOCKER_IMAGE &&
                                docker stop $DOCKER_CONTAINER || true &&
                                docker rm $DOCKER_CONTAINER || true &&
                                docker run -d --name $DOCKER_CONTAINER -p 5000:5000 $DOCKER_IMAGE
                            EOF
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            mail to: 'vijay.rhce@gmail.com',
                 subject: "Build Successful: ${env.JOB_NAME}",
                 body: "The build was successful!"
        }
        failure {
            mail to: 'vijay.rhce@gmail.com',
                 subject: "Build Failed: ${env.JOB_NAME}",
                 body: "The build has failed. Check Jenkins for details."
        }
    }
}