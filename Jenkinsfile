pipeline {
    agent any

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }
        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }
        stage('Run Tests') {
            steps {
                sh '''
                npx html-validate index.html
                npx stylelint "css/**/*.css"
                '''
            }
        }
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t tourism-website:${BUILD_NUMBER} .'
                sh 'docker tag tourism-website:${BUILD_NUMBER} tourism-website:latest'
            }
        }
        stage('Deploy Container') {
            steps {
                sh '''
                docker stop tourism-container || true
                docker rm tourism-container || true
                docker run -d -p 8000:80 --name tourism-container tourism-website:latest
                '''
            }
        }
    }
    post {
        success {
            echo 'All Test Cases Passed and Application Deployed Successfully!'
        }
        failure {
            echo 'Build or Tests Failed. Please Check the Logs.'
        }
    }
}
