pipeline {
    agent any

    stages {
        stage('Build Docker Image') {
            steps {
                script {
                    configFileProvider([configFile(fileId: 'e23cf428-e733-4803-8c2a-8427627bbc3f', targetLocation: '.env')]) {
                        sh 'docker build -t todo-list-app .'
                    }
                }
            }
        }

        stage('Push to DockerHub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKERHUB_USER', passwordVariable: 'DOCKERHUB_PASS')]) {
                        sh """
                            docker login -u '${DOCKERHUB_USER}' -p '${DOCKERHUB_PASS}'
                            docker tag todo-list-app ${DOCKERHUB_USER}/todo-list-app:latest
                            docker push ${DOCKERHUB_USER}/todo-list-app:latest
                        """
                    }
                }
            }
        }

        stage('Deploy to Development') {
            steps {
                script {
                    sh """
                        docker rm -f todo-list-dev
                        docker run -d -p 8001:8000 --name todo-list-dev efranca/todo-list-app:latest
                    """
                }
            }
        }

    }
}

