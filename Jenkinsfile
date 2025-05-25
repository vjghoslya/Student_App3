pipeline {
    agent any

    environment {
        VENV_DIR = 'venv'
        DOCKER_IMAGE = 'studentapp-image:latest'
        DOCKER_CONTAINER = 'studentapp-container'
        STAGING_SERVER = '192.168.188.142'
        REMOTE_USER = 'ubuntu'
    }

    stages {
        stage('Clone Repo') {
            steps {
                git branch: 'main', url: 'https://github.com/vjghoslya/Student_App3.git'
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

        stage('Run Tests  ..... ') {
            steps {
                sh '''
		    . $VENV_DIR/bin/activate
                    pytest test_app.py
                '''
            }
        }

        stage('Dockerize Application .....') {
            when {
                expression { currentBuild.result == null || currentBuild.result == 'SUCCESS' }
            }
            steps {
                sh "docker build -t $DOCKER_IMAGE ."
            }
        }

        stage('Deploy to Staging Server $STAGING_SERVER .') {
            when {
                expression { currentBuild.result == null || currentBuild.result == 'SUCCESS' }
            }
            steps {
                sh '''
                    docker save $DOCKER_IMAGE | bzip2 | ssh -o StrictHostKeyChecking=no $REMOTE_USER@$STAGING_SERVER 'bunzip2 | docker load'
                    ssh $REMOTE_USER@$STAGING_SERVER '
                        docker stop $DOCKER_CONTAINER || true &&
                        docker rm $DOCKER_CONTAINER || true &&
                        docker run -d --name $DOCKER_CONTAINER -p 8000:8000 yourapp-image:latest
                    '
                '''
            }
        }
    }

    post {
        success {
            mail to: 'vijay.rhce@gmail.com',
                 subject: "Build Successful : ${env.JOB_NAME}",
                 body: "The build was successful!"
        }
        failure {
            mail to: 'vijay.rhce@gmail.com',
                 subject: "Build Failed: ${env.JOB_NAME}",
                 body: "The build has failed. Check Jenkins for details."
        }
    }
}
