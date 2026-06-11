pipeline {
    agent any

    environment {
        DOCKER_CREDENTIALS = credentials('dockerhub-credentials')
        DOCKER_USERNAME    = "${DOCKER_CREDENTIALS_USR}"
        DOCKER_PASSWORD    = "${DOCKER_CREDENTIALS_PSW}"
        IMAGE_NAME         = "sherinputhukkudy/my-app"
        GIT_REPO           = "https://github.com/Sherinp563/my-app-k8s.git"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Set Image Tag') {
            steps {
                script {
                    env.IMAGE_TAG = sh(
                        script: 'git rev-parse --short HEAD',
                        returnStdout: true
                    ).trim()
                    echo "Building image: ${IMAGE_NAME}:${IMAGE_TAG}"
                }
            }
        }

        stage('Build Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }

        stage('Push Image') {
            steps {
                sh """
                    echo ${DOCKER_PASSWORD} | docker login -u ${DOCKER_USERNAME} --password-stdin
                    docker push ${IMAGE_NAME}:${IMAGE_TAG}
                """
            }
        }

        stage('Update Manifest') {
            steps {
                sh """
                    # Clone the manifests repo
                    git clone ${GIT_REPO} k8s-repo
                    cd k8s-repo

                    # Update the image tag in deployment.yaml
                    sed -i 's|image: ${IMAGE_NAME}:.*|image: ${IMAGE_NAME}:${IMAGE_TAG}|' k8s/deployment.yaml

                    # Commit and push the change
                    git config user.email "jenkins@ci.com"
                    git config user.name "Jenkins"
                    git add k8s/deployment.yaml
                    git commit -m "Update image tag to ${IMAGE_TAG}"
                    git push
                """
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline succeeded — ${IMAGE_NAME}:${IMAGE_TAG} deployed"
        }
        failure {
            echo "❌ Pipeline failed — check logs above"
        }
    }
}
