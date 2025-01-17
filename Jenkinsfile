pipeline {
    
    agent any

    environment {
        DEPLOY_ENV = 'production'
    }
    
    stages {

        stage('Check ENV') {
            steps {
                script {
                    sh 'echo "DEPLOY_ENV=${DEPLOY_ENV}"'
                }
            }
        } 
        
        stage('Build Docker Image') {
            steps {
                script {
                    configFileProvider([configFile(fileId: '799e1db3-3681-4872-98e9-b1fcd977112f', targetLocation: '.env')]) {
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
                            docker tag todo-list-app ${DOCKERHUB_USER}/todo-list-app:${BUILD_NUMBER}
                            docker push ${DOCKERHUB_USER}/todo-list-app:${BUILD_NUMBER}
                        """
                    }
                }
            }
        }
        
        stage('Scan Docker Image') {
            steps {
                script {
                    def trivyOutput = sh(script: "trivy image todo-list-app", returnStdout: true).trim()
                    println trivyOutput

                    if (trivyOutput.contains("Total: 0")) {
                        echo "No vulnerabilities found in the Docker image."
                    } else {
                        echo "Vulnerabilities found in the Docker image."
                    }
                }
            }
        }     

        stage('Deploy to Development') {
            steps {
                script {
                    sh """
                        docker rm -f todo-list-dev
                        docker run -d -p 8001:8000 --name todo-list-dev efranca/todo-list-app:${BUILD_NUMBER}
                    """
                }
            }
        }

        stage('Deploy to Production') {
            when {
                environment name: 'DEPLOY_ENV', value: 'production'
            }
            steps {
                script {
                    def userInput = input(message: "Deploy to Production? (yes/no)", ok: 'Deploy',
                    parameters: [choice(choices: ['yes', 'no'], description: 'Approve deployment?', name: 'decision')])
                    if (userInput == 'yes') {
                        withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKERHUB_USER', passwordVariable: 'DOCKERHUB_PASS')]) {
                            sh """
                                docker login -u '${DOCKERHUB_USER}' -p '${DOCKERHUB_PASS}'
                                docker pull ${DOCKERHUB_USER}/todo-list-app:latest 
                                docker rm -f todo-list-app-prod
                                docker run -d --name todo-list-app-prod -p 8000:8000 ${DOCKERHUB_USER}/todo-list-app:latest
                            """
                        }
                    } else {
                        currentBuild.result = 'ABORTED'
                        error('Deploy to production cancelled')
                    }
                }
            }
        }
    }
}
