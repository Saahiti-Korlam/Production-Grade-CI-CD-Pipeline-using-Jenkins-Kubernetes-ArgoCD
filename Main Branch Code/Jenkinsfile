pipeline {
  agent any

  options {
    disableConcurrentBuilds()
  }

  environment {
    IMAGE_NAME = "sahichoco123/multistage-flask-app"
    GIT_USER = "Saahiti-Korlam"
    GIT_EMAIL = "saahiti0699@gmail.com"
  }

  stages {

    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build and Push Docker Image') {
      when { branch 'main' }
      steps {
        script {
          env.IMAGE_TAG = "build-${env.BUILD_NUMBER}"

          withCredentials([usernamePassword(
            credentialsId: 'dockerhub_cred',
            usernameVariable: 'DOCKERHUB_USERNAME',
            passwordVariable: 'DOCKERHUB_PASSWORD'
          )]) {

            sh """
              docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
              echo "$DOCKERHUB_PASSWORD" | docker login -u "$DOCKERHUB_USERNAME" --password-stdin
              docker push ${IMAGE_NAME}:${IMAGE_TAG}
            """
          }
        }
      }
    }

    stage('Deploy to Kubernetes manifest file') {
      when { branch 'main' }
      steps {
        script {
          withCredentials([usernamePassword(
            credentialsId: 'github_cred',
            usernameVariable: 'GITHUB_USERNAME',
            passwordVariable: 'GITHUB_TOKEN'
          )]) {

            sh """
              set -e
              git config user.name "${GIT_USER}"
              git config user.email "${GIT_EMAIL}"

              git fetch origin
              git checkout main
              git reset --hard origin/main

              sed -i "s|image:.*|image: ${IMAGE_NAME}:${IMAGE_TAG}|" kubernetes/deployment.yaml

              git add kubernetes/deployment.yaml
              git diff --cached --quiet || git commit -m "Update deployment with image tag ${IMAGE_TAG}"
              git push https://${GITHUB_USERNAME}:${GITHUB_TOKEN}@github.com/Saahiti-Korlam/multi-branch-test.git main
            """
          }
        }
      }
    }
  }
}
