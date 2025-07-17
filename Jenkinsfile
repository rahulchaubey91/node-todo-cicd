pipeline {
    agent any

    environment {
        IMAGE_NAME = "node-app-test-new"
    }

    stages {

        stage("Code Checkout") {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/master']],
                    extensions: [[$class: 'CloneOption', depth: 1, shallow: true]],
                    userRemoteConfigs: [[url: 'https://github.com/rahulchaubey91/node-todo-cicd.git']]
                ])
                echo 'Code clone ho gaya (shallow)'
            }
        }

        stage("Build & Test") {
            steps {
                sh "docker build --cache-from ${env.IMAGE_NAME} -t ${env.IMAGE_NAME} ."
                echo 'Build ho gaya (with cache)'
            }
        }

        stage("Scan Image") {
            steps {
                echo 'Trivy Docker container se image scan ho raha hai...'
                sh """
                    docker run --rm \
                        -v /var/run/docker.sock:/var/run/docker.sock \
                        -v \$HOME/.cache:/root/.cache \
                        aquasec/trivy image --severity HIGH,CRITICAL ${env.IMAGE_NAME}
                """
                echo 'Trivy scan complete'
            }
        }

        stage("Push to DockerHub") {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: "dockerHub",
                        usernameVariable: "dockerHubUser",
                        passwordVariable: "dockerHubPass"
                    )
                ]) {
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
