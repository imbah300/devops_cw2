pipeline {
    agent any

    environment {
        IMAGE_NAME = "imbah300/cw2-server"
        IMAGE_TAG = "1.0"
	DOCKER_CRED = credentials("docker")
    }

    triggers {
        githubPush()
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/imbah300/devops_cw2.git'
            }
        }

        stage('Build Image') {
            steps {
                sh """
                    sudo docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                """
            }
        }

        stage('Test Container') {
            steps {
                sh """
                    docker run -d --name test -p 8081:8081 ${IMAGE_NAME}:${IMAGE_TAG}
                    sleep 8
                    docker exec test echo "OK"
                    docker stop test
                    docker rm test
                """
            }
        }
	
	stage('Docker Login') {
		steps {
			sh 'echo $DOCKER_CRED | docker login -u imbah300 $DOCKER_CRED_PSW --password-stdin'
		}
	}

        stage('Push DockerHub') {
            steps {
                sh """
                    docker push ${IMAGE_NAME}:${IMAGE_TAG}
                """
            }
        }

        stage('Deploy via Remote Ansible') {
            steps {
                sh """
                    ssh -o StrictHostKeyChecking=no ubuntu@ec2-98-88-250-195.compute-1.amazonaws.com \\
                    'ansible-playbook k8s.yml'
                """
            }
        }
    }
}
