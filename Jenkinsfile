pipeline {
    agent any

    options {
        skipDefaultCheckout(true) // so Jenkins doesn’t auto-checkout before clean
    }    

    tools {
	nodejs 'NodeJS'
    }

    stages {
	stage('Cleanup') {
            steps {
                cleanWs()         
            }
        }
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
                npx html-validator --file=index.html --format=text
                docker build -t tourism-test:latest .
		docker run -d -p 9000:80 --name tourism-test-container tourism-test:latest
		sleep 10
		curl -f http://localhost:9000 || exit 1
		npx broken-link-checker http://localhost:9000 --recursive --exclude-external
		docker stop tourism-test-container
                docker rm tourism-test-container
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
