pipeline {
    agent none
    environment {
        DOCKER_ID = "p0l1na"
        DOCKER_TAG = "v.${BUILD_ID}.0"
        DOCKER_PASS = credentials("DOCKER_HUB_PASS")
    }
    stages {
        stage('Docker build images') {
            parallel {
                stage ('main/cast-service') {
                    environment
                    {
                        DOCKER_IMAGE = "jenkins_fastapi_casts"
                    }
                    steps {
                        script(' Docker Build') {
                            sh '''
                            docker rm -f jenkins
                            docker build -t $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG .
                            sleep 6
                            '''
                        }
                        script('Docker run') {
                                sh '''
                                docker run -d -p 80:80 --name jenkins $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG
                                sleep 10
                                '''
                        }
                        script('Test Acceptance') {
                                sh '''
                                curl localhost
                                '''
                        }
                        script('Docker Push') {
                                sh '''
                                'env'
                                docker login -u $DOCKER_ID -p $DOCKER_PASS
                                docker push $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG
                                '''
                        }
                    }
                }
                stage ('main/movie-service') {
                    environment
                    {
                        DOCKER_IMAGE = "jenkins_fastapi_movies"
                    }
                    steps {
                        script(' Docker Build') {
                            sh '''
                            docker rm -f jenkins
                            docker build -t $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG .
                            sleep 6
                            '''
                        }
                        script('Docker run') {
                            sh '''
                            docker run -d -p 80:80 --name jenkins $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG
                            sleep 10
                            '''
                        }
                        script('Test Acceptance') {
                            sh '''
                            curl localhost
                            '''
                        }
                        script('Docker Push') {
                            sh '''
                            'env'
                            docker login -u $DOCKER_ID -p $DOCKER_PASS
                            docker push $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG
                            '''
                        }
                    }
                }
            }
        }
        stage('Dev deployment') {
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                script {
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    ls
                    cat $KUBECONFIG > .kube/config
                    cp fastapi/values.yaml values.yml
                    cat values.yml
                    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                    helm upgrade --install app fastapi --values=values.yml --namespace dev
                    '''
                }
            }
        }
        stage('QA deployment') {
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                script {
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    ls
                    cat $KUBECONFIG > .kube/config
                    cp fastapi/values.yaml values.yml
                    cat values.yml
                    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                    helm upgrade --install app fastapi --values=values.yml --namespace qa
                    '''
                }
            }
        }
        stage('Staging deployment') {
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                script {
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    ls
                    cat $KUBECONFIG > .kube/config
                    cp fastapi/values.yaml values.yml
                    cat values.yml
                    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                    helm upgrade --install app fastapi --values=values.yml --namespace staging
                    '''
                }
            }
        }
        stage('Prod deployment') {
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                timeout(time: 15, unit: "MINUTES") {
                    input message: 'Do you want to deploy in production ?', ok: 'Yes'
                }
                script {
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    ls
                    cat $KUBECONFIG > .kube/config
                    cp fastapi/values.yaml values.yml
                    cat values.yml
                    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                    helm upgrade --install app fastapi --values=values.yml --namespace prod
                    '''
                }
            }
        }
    }
}
