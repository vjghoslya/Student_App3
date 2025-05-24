pipeline {
    agent any

    stages {
        stage('Clone Repo') {
            steps {
                git 'https://github.com/vjghoslya/Student_App3.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'pip install -r requirements.txt'
            }
        }

        stage('Run Tests') {
            steps {
                sh 'pytest test_app.py --maxfail=1 --disable-warnings -q'
            }
        }

        stage('Run Flask App') {
            steps {
                sh 'docker run -d 5000:5000 python:3.10 nohup python app.py &'
            }
        }
    }
    post {
        success {
            mail to: 'you@example.com',
                 subject: "Build Successful: ${env.JOB_NAME}",
                 body: "The build was successful!"
        }
        failure {
            mail to: 'you@example.com',
                 subject: "Build Failed: ${env.JOB_NAME}",
                 body: "The build has failed. Check Jenkins for details."
        }
    }
}
