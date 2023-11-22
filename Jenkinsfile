pipeline {
    agent none
    environment {
        DOCKER_ID = "p0l1na"
        DOCKER_TAG = "v.${BUILD_ID}.0"
    }
    stages {
        parallel {
            stage ('Fastapi casts') {
                agent {
                    docker {
                        image = "jenkins_fastapi_casts"
                    }
                }
                steps('Docker build') {
                    script {
                        sh '''
                        docker rm -f jenkins
                        docker build -t $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG .
                        sleep 6
                        '''
                    }
                }
                steps('Docker run') {
                    script {
                        sh '''
                        docker run -d -p 80:80 --name jenkins $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG
                        sleep 10
                        '''
                    }
                }
                steps('Test Acceptance') {
                    script {
                        sh '''
                        curl localhost
                        '''
                    }
                }
                steps('Docker Push') {
                    environment
                    {
                        DOCKER_PASS = credentials("DOCKER_HUB_PASS")
                    }
                    script {
                        sh '''
                        docker login -u $DOCKER_ID -p $DOCKER_PASS
                        docker push $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG
                        '''
                    }
                }

            },
            stage ('Fastapi movies') {
                agent {
                    docker {
                        image = "jenkins_fastapi_movies"
                    }
                }
                steps(' Docker Build') {
                    script {
                        sh '''
                        docker rm -f jenkins
                        docker build -t $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG .
                        sleep 6
                        '''
                    }
                }
                steps('Docker run') {
                    script {
                        sh '''
                        docker run -d -p 80:80 --name jenkins $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG
                        sleep 10
                        '''
                    }
                }
                steps('Test Acceptance') {
                    script {
                        sh '''
                        curl localhost
                        '''
                    }
                }
                steps('Docker Push') {
                    environment
                    {
                        DOCKER_PASS = credentials("DOCKER_HUB_PASS")
                    }
                    script {
                        sh '''
                        docker login -u $DOCKER_ID -p $DOCKER_PASS
                        docker push $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG
                        '''
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
