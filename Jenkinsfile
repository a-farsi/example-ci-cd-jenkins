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
                        script {
                            sh '''
                            rm -Rf .kube
                            mkdir .kube
                            ls
                            cat $KUBECONFIG > .kube/config
                            
                            # Copie du fichier values.yaml pour modification
                            cp charts/values.yaml values.yml
                            cat values.yml

                            # Mise à jour du tag d'image dans values.yml
                            sed -i "s+tag:.*+tag: ${DOCKER_TAG}+g" values.yml
                            
                            # Déploiement avec Helm sur le namespace dev
                            helm upgrade --install my-app charts/ --values=values.yml --namespace dev --create-namespace
                            '''
                        }
                    }
                }
                stage('Deploy to QA') {
                    steps {
                        script {
                            sh '''
                            rm -Rf .kube
                            mkdir .kube
                            ls
                            cat $KUBECONFIG > .kube/config

                            # Copie du fichier values.yaml pour modification
                            cp charts/values.yaml values.yml
                            cat values.yml

                            # Mise à jour du tag d'image dans values.yml
                            sed -i "s+tag:.*+tag: ${DOCKER_TAG}+g" values.yml

                            # Déploiement avec Helm sur le namespace dev
                            helm upgrade --install my-app charts/ --values=values.yml --namespace qa --create-namespace
                            '''
                        }
                    }
                }
                stage('Deploy to Staging') {
                    steps {
                        script {
                            sh '''
                            rm -Rf .kube
                            mkdir .kube
                            ls
                            cat $KUBECONFIG > .kube/config

                            # Copie du fichier values.yaml pour modification
                            cp charts/values.yaml values.yml
                            cat values.yml

                            # Mise à jour du tag d'image dans values.yml
                            sed -i "s+tag:.*+tag: ${DOCKER_TAG}+g" values.yml

                            # Déploiement avec Helm sur le namespace dev
                            helm upgrade --install my-app charts/ --values=values.yml --namespace staging --create-namespace
                            '''
                        }
                    }
                }
            }
        }
        
        stage('Check the current branch') {
            steps {
                sh 'echo "Current branch is ${GIT_BRANCH}"'
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
                sh '''
                rm -Rf .kube
                mkdir .kube
                ls
                cat $KUBECONFIG > .kube/config

                # Copie du fichier values.yaml pour modification
                cp charts/values.yaml values.yml
                cat values.yml

                # Mise à jour du tag d'image dans values.yml
                sed -i "s+tag:.*+tag: ${DOCKER_TAG}+g" values.yml

                # Déploiement avec Helm sur le namespace dev
                helm upgrade --install my-app charts/ --values=values.yml --namespace prod --create-namespace
                '''
            }
        }
    }
}
