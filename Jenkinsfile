pipeline {
    agent any

    environment {
        IMAGE_NAME = "student-web"
        APP_SERVER = credentials('app-server-ip')
        SSH_KEY = credentials('app-server-ssh-key')
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Dohvaćanje koda s GitHuba...'
                checkout scm
            }
        }

        stage('Build') {
            steps {
                echo 'Izgradnja Docker imagea...'
                sh 'docker build -t ${IMAGE_NAME}:latest .'
                sh 'docker images | grep ${IMAGE_NAME}'
            }
        }

        stage('Test') {
            steps {
                echo 'Pokretanje testova...'
                sh '''
                    docker run -d --name test-container -p 8081:80 ${IMAGE_NAME}:latest
                    sleep 5
                    curl -f http://localhost:8081 | grep -i "Antonio Šajić"
                    docker stop test-container
                    docker rm test-container
                '''
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deployment na app-server...'
                sh '''
                    docker save ${IMAGE_NAME}:latest | \
                    ssh -i ${SSH_KEY} -o StrictHostKeyChecking=no \
                    ubuntu@${APP_SERVER} \
                    "docker load"

                    ssh -i ${SSH_KEY} -o StrictHostKeyChecking=no \
                    ubuntu@${APP_SERVER} \
                    "mkdir -p /home/ubuntu/app"

                    scp -i ${SSH_KEY} -o StrictHostKeyChecking=no \
                    docker-compose.yml \
                    ubuntu@${APP_SERVER}:/home/ubuntu/app/docker-compose.yml

                    ssh -i ${SSH_KEY} -o StrictHostKeyChecking=no \
                    ubuntu@${APP_SERVER} \
                    "cd /home/ubuntu/app && docker-compose down && docker-compose up -d"
                '''
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline završen uspješno!'
        }
        failure {
            echo '❌ Pipeline nije uspio!'
        }
    }
}
