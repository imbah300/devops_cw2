pipeline {
    agent any

    environment {
        IMAGE_NAME = "imbah300/cw2-server"
        IMAGE_TAG = "1.0"
        PROD_SERVER = "ubuntu@ec2-98-88-250-195.compute-1.amazonaws.com"
        PROD_PLAYBOOK = "k8s.yml"
    }

    triggers {
        githubPush()
    }

    stages {

        stage('Checkout Source Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/imbah300/devops_cw2.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                    docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                    docker tag ${IMAGE_NAME}:${IMAGE_TAG}
                """
            }
        }

        stage('Test Container Launch') {
            steps {
                sh """
                    docker run -d --name test-container -p 8081:8081 ${IMAGE_NAME}:${IMAGE_TAG}
                    sleep 10
                    docker ps | grep test-container
                    docker exec test-container echo "Container started successfully"
                    docker stop test-container
                    docker rm test-container
                """
            }
        }

        stage('Push to DockerHub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh """
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push ${IMAGE_NAME}:${IMAGE_TAG}
                    """
                }
            }
        }

        stage('Deploy to Production via Ansible') {
            steps {
                sh """
                    ssh -o StrictHostKeyChecking=no ${PROD_SERVER} \
                    'ansible-playbook ${PROD_PLAYBOOK}'
                """
            }
        }
    }

    post {
        success {
            echo 'Build, test, push, and deployment completed successfully.'
        }

        failure {
            echo 'Pipeline failed. Check the logs above for details.'
        }

        always {
            sh 'docker system prune -f || true'
        }
    }
}
