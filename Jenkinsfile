pipeline {

    agent any

    environment {
        IMAGE_NAME = "kiranthorat8419/nodejs-app"
        TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                url: 'https://github.com/Kiran-krt/practice_k8s.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                docker build -t $IMAGE_NAME:$TAG .
                """
            }
        }

        stage('Push Docker Image') {
            steps {

                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {

                    sh """
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin

                    docker push $IMAGE_NAME:$TAG
                    """
                }
            }
        }

        stage('Update Manifest Repo') {

            steps {

                sh """

                rm -rf k8s-manifests

                git clone https://github.com/Kiran-krt/k8s-manifests.git

                cd k8s-manifests

                sed -i 's|image:.*|image: $IMAGE_NAME:$TAG|g' deployment.yaml

                git config user.email "thoratkiran2122@gmail.com"

                git config user.name "Kiran-krt"

                git add deployment.yaml

                git commit -m "Updated image $TAG"

                git push

                """
            }
        }
    }

    post {

        success {
            echo 'Build completed'
        }

        failure {
            echo 'Build failed'
        }
    }
}