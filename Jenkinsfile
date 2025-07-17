pipeline {
    agent any

    environment {
        IMAGE_NAME = "node-app-test-new"
    }

    stages {

        stage("Code Checkout") {
            steps {
                checkout([$class: 'GitSCM',
                    branches: [[name: '*/master']],
                    extensions: [[$class: 'CloneOption', depth: 1, shallow: true]],
                    userRemoteConfigs: [[url: 'https://github.com/rahulchaubey91/node-todo-cicd.git']]
                ])
                echo 'Code clone ho gaya (shallow)'
            }
        }

        stage("Build & Test") {
            steps {
                sh "docker build -t ${env.IMAGE_NAME} ."
                echo 'Build ho gaya (with cache)'
            }
        }

        stage("Scan Image") {
            steps {
                echo 'Image scan (placeholder)'
                // Optionally use Trivy or Anchore here
            }
        }

        stage("Push to DockerHub") {
            steps {
                withCredentials([usernamePassword(credentialsId: "dockerHub", passwordVariable: "dockerHubPass", usernameVariable: "dockerHubUser")]) {
                    sh """
                        docker login -u ${dockerHubUser} -p ${dockerHubPass}
                        docker tag ${IMAGE_NAME}:latest ${dockerHubUser}/${IMAGE_NAME}:latest
                        docker push ${dockerHubUser}/${IMAGE_NAME}:latest
                    """
                    echo 'Docker image push ho gaya'
                }
            }
        }

        stage("Deploy") {
            steps {
                sh """
                    docker-compose pull
                    docker-compose up -d --build
                """
                echo 'Deployment complete (no downtime)'
            }
        }
    }
}
