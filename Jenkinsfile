pipeline {
    agent any

    environment {
        DOCKERHUB_REPO = "omkarmemane9"
        FRONTEND_IMAGE = "easycrud-frontend"
        BACKEND_IMAGE  = "easycrud-backend"
        VERSION        = "${BUILD_NUMBER}"
    }

    stages {

        stage('Clone Repo') {
            steps {
                git branch: 'main', url: 'https://github.com/DevBash9/EasyCRUD-Jenkins.git'
            }
        }

        stage('Build Images') {
            steps {
                sh """
                docker build -t $DOCKERHUB_REPO/$FRONTEND_IMAGE:$VERSION ./frontend
                docker build -t $DOCKERHUB_REPO/$FRONTEND_IMAGE:latest ./frontend
                
                docker build --no-cache -t $DOCKERHUB_REPO/$BACKEND_IMAGE:$VERSION ./backend
                docker build --no-cache -t $DOCKERHUB_REPO/$BACKEND_IMAGE:latest ./backend
                """
            }
        }

        stage('Docker Hub Login') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-cred',
                    usernameVariable: 'USERNAME',
                    passwordVariable: 'PASSWORD'
                )]) {
                    sh 'echo $PASSWORD | docker login -u $USERNAME --password-stdin'
                }
            }
        }

        stage('Push Images') {
            steps {
                sh """
                docker push $DOCKERHUB_REPO/$FRONTEND_IMAGE:$VERSION
                docker push $DOCKERHUB_REPO/$FRONTEND_IMAGE:latest
                
                docker push $DOCKERHUB_REPO/$BACKEND_IMAGE:$VERSION
                docker push $DOCKERHUB_REPO/$BACKEND_IMAGE:latest
                """
            }
        }

        stage('Deploy Containers') {
            steps {
                sh """
                docker rm -f easycrud-frontend || true
                docker rm -f easycrud-backend || true

                docker run -d --name easycrud-frontend \
                  -p 80:80 \
                  $DOCKERHUB_REPO/$FRONTEND_IMAGE:latest

                docker run -d --name easycrud-backend \
                  -p 8082:8080 \
                  -e SPRING_DATASOURCE_PASSWORD=redhat123 \
                  $DOCKERHUB_REPO/$BACKEND_IMAGE:latest
                """
            }
        }

        stage('Verify Deployment') {
            steps {
                sh "docker ps"
            }
        }
    }
}
