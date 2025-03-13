pipeline {
    environment {
        DOCKER_ID = "afarsi"
        DOCKER_CAST_IMAGE = "cast-service"
        DOCKER_MOVIE_IMAGE = "movie-service"
        DOCKER_TAG = "v.${BUILD_ID}.0"
    }
    agent any
    stages {
        stage('Checkout') {
            steps {
                script {
                    git 'https://github.com/a-farsi/example-ci-cd-jenkins.git'
                }
            }
        }

        stage('Build Docker Images') {
            parallel {
                stage('Build Cast Service') {
                    steps {
                        sh "docker build -t $DOCKER_ID/$DOCKER_CAST_IMAGE:$DOCKER_TAG cast-service/"
                    }
                }
                stage('Build Movie Service') {
                    steps {
                        sh "docker build -t $DOCKER_ID/$DOCKER_MOVIE_IMAGE:$DOCKER_TAG movie-service/"
                    }
                }
            }
        }

        stage('Push to DockerHub') {
            environment {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS")
            }
            steps {
                sh """
                docker login -u $DOCKER_ID -p $DOCKER_PASS
                docker push $DOCKER_ID/$DOCKER_CAST_IMAGE:$DOCKER_TAG
                docker push $DOCKER_ID/$DOCKER_MOVIE_IMAGE:$DOCKER_TAG
                """
            }
        }

        stage('Deploy to Kubernetes') {
            environment {
                KUBECONFIG = credentials("config")
            }
            parallel {
                stage('Deploy to Dev') {
                    steps {
                        sh """
                        kubectl config use-context default
                        kubectl apply -f charts/ -n dev
                        """
                    }
                }
                stage('Deploy to QA') {
                    steps {
                        sh """
                        kubectl apply -f charts/ -n qa
                        """
                    }
                }
                stage('Deploy to Staging') {
                    steps {
                        sh """
                        kubectl apply -f charts/ -n staging
                        """
                    }
                }
            }
        }

        stage('Deploy to Production') {
            when {
                branch 'master'
            }
            steps {
                timeout(time: 15, unit: "MINUTES") {
                    input message: 'Do you want to deploy in production?', ok: 'Deploy'
                }
                sh """
                kubectl apply -f charts/ -n prod
                """
            }
        }
    }
}

