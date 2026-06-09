pipeline {

    agent any

    triggers {
        githubPush()
    }

    environment {
        IMAGE_NAME = "kiranthorat8419/nodejs-app"
        TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout Source Code') {
            steps {
                git(
                    branch: 'main',
                    url: 'https://github.com/Kiran-krt/practice_k8s.git'
                )
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                    docker build -t ${IMAGE_NAME}:${TAG} .
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
                        echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin

                        docker push ${IMAGE_NAME}:${TAG}

                        docker logout
                    """
                }
            }
        }

        stage('Update Manifest Repository') {
            steps {

                withCredentials([
                    usernamePassword(
                        credentialsId: 'github-creds',
                        usernameVariable: 'GIT_USER',
                        passwordVariable: 'GIT_TOKEN'
                    )
                ]) {

                    sh """
                        rm -rf k8s-manifests

                        git clone https://\$GIT_USER:\$GIT_TOKEN@github.com/Kiran-krt/k8s-manifests.git

                        cd k8s-manifests

                        sed -i 's|image:.*|image: ${IMAGE_NAME}:${TAG}|g' deployment.yaml

                        git config user.email "thoratkiran2122@gmail.com"
                        git config user.name "Kiran-krt"

                        git add deployment.yaml

                        if git diff --cached --quiet; then
                            echo "No changes detected in deployment.yaml"
                        else
                            git commit -m "Update image tag to ${TAG}"
                            git push origin main
                        fi
                    """
                }
            }
        }
    }

    post {

        success {
            echo 'Pipeline completed successfully'
        }

        failure {
            echo 'Pipeline failed'
        }

        always {
            cleanWs()
        }
    }
}