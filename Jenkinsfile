pipeline {
    agent { label 'docker-agent' }

    environment {
        IMAGE_NAME = "tchelet/cicd"
        IMAGE_TAG = "${BUILD_NUMBER}"
        GIT_CREDENTIALS = '9955446b-dc74-40aa-b7e3-bfa9e8071c0a'
        MANIFESTS_REPO = 'https://github.com/Tchelet/django-app-ci-cd-manifests.git'
        MANIFESTS_BRANCH = 'main'
        DOCKERHUB_CREDENTIALS = 'ea6ea248-89fc-40c6-8cbb-a2fa518887cc'  
    }

    stages {
        stage('Checkout Application Code') {
            steps {
                git credentialsId: env.GIT_CREDENTIALS, url: 'https://github.com/Tchelet/django-app-ci-cd.git', branch: 'main'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .'
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: env.DOCKERHUB_CREDENTIALS, passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                    sh '''
                    echo "${DOCKER_PASSWORD}" | docker login -u "${DOCKER_USERNAME}" --password-stdin
                    docker push ${IMAGE_NAME}:${IMAGE_TAG}
                    '''
                }
            }
        }

        stage('Update Manifests and Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: env.GIT_CREDENTIALS, passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
                    sh '''
                    git clone https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/Tchelet/django-app-ci-cd-manifests.git
                    cd django-app-ci-cd-manifests
                    sed -i "s|image:.*|image: ${IMAGE_NAME}:${IMAGE_TAG}|g" deploy.yaml
                    git add deploy.yaml
                    git commit -m "Update image tag to ${IMAGE_TAG}"
                    git push origin ${MANIFESTS_BRANCH}
                    '''
                }
            }
        }
    }
}
