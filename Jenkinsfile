pipeline {
    agent any

    environment {
        IMAGE_NAME = "tchelet/cicd"
        IMAGE_TAG = "${BUILD_NUMBER}"
        REGISTRY_CREDENTIALS = 'ea6ea248-89fc-40c6-8cbb-a2fa518887cc	'  // Docker Hub credentials ID
        GIT_CREDENTIALS = '9955446b-dc74-40aa-b7e3-bfa9e8071c0a'          // GitHub credentials ID
        MANIFESTS_REPO = 'https://github.com/Tchelet/django-app-ci-cd-manifests.git'
        MANIFESTS_BRANCH = 'main'
    }

    stages {
        stage('Checkout Application Code') {
            steps {
                git credentialsId: env.GIT_CREDENTIALS, url: 'https://github.com/Tchelet/django-app-ci-cd.git', branch: 'main'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${env.IMAGE_NAME}:${env.IMAGE_TAG} ."
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: env.REGISTRY_CREDENTIALS, passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                        sh """
                        echo "${DOCKER_PASSWORD}" | docker login -u "${DOCKER_USERNAME}" --password-stdin
                        docker push ${env.IMAGE_NAME}:${env.IMAGE_TAG}
                        """
                    }
                }
            }
        }

        stage('Update Manifests and Push') {
            steps {
                script {
                    // Clone manifests repo
                    git credentialsId: env.GIT_CREDENTIALS, url: env.MANIFESTS_REPO, branch: env.MANIFESTS_BRANCH
                    // Update the image tag in deployment manifest
                    sh """
                    sed -i "s|image: ${env.IMAGE_NAME}:.*|image: ${env.IMAGE_NAME}:${env.IMAGE_TAG}|g" deploy.yaml
                    git add deploy.yaml
                    git commit -m "Update image tag to ${env.IMAGE_TAG}"
                    git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/Tchelet/django-app-ci-cd-manifests.git HEAD:${env.MANIFESTS_BRANCH}
                    """
                }
            }
        }
    }
}
