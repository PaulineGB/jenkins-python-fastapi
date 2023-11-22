pipeline {
    agent any
    environment {
        DOCKER_ID = "p0l1na"
        DOCKER_TAG = "v.${BUILD_ID}.0"
    }
    stages {
        stage(' Docker Build') {
            steps {
                environment {
                    DOCKER_IMAGE = "jenkins_fastapi_casts"
                }
                script {
                    sh '''
                    docker rm -f jenkins
                    docker build -t $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG .
                    sleep 6
                    '''
                }
            }
            steps {
                environment {
                    DOCKER_IMAGE = "jenkins_fastapi_movies"
                }
                script {
                    sh '''
                    docker rm -f jenkins
                    docker build -t $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG .
                    sleep 6
                    '''
                }
            }
        }
        stage('Docker run') {
            steps {
                environment {
                    DOCKER_IMAGE = "jenkins_fastapi_casts"
                }
                script {
                    sh '''
                    docker run -d -p 80:80 --name jenkins $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG
                    sleep 10
                    '''
                }
            }
            steps {
                environment {
                    DOCKER_IMAGE = "jenkins_fastapi_movies"
                }
                script {
                    sh '''
                    docker run -d -p 80:80 --name jenkins $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG
                    sleep 10
                    '''
                }
            }
        }
        stage('Test Acceptance') {
            steps {
                environment {
                    DOCKER_IMAGE = "jenkins_fastapi_casts"
                }
                script {
                    sh '''
                    curl localhost
                    '''
                }
            }
            steps {
                environment {
                    DOCKER_IMAGE = "jenkins_fastapi_movies"
                }
                script {
                    sh '''
                    curl localhost
                    '''
                }
            }
        }
        stage('Docker Push') {
            environment
            {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS")
            }
            steps {
                environment {
                    DOCKER_IMAGE = "jenkins_fastapi_casts"
                }
                script {
                    sh '''
                    docker login -u $DOCKER_ID -p $DOCKER_PASS
                    docker push $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG
                    '''
                }
            }
            steps {
                environment {
                    DOCKER_IMAGE = "jenkins_fastapi_movies"
                }
                script {
                    sh '''
                    docker login -u $DOCKER_ID -p $DOCKER_PASS
                    docker push $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG
                    '''
                }
            }
        }
        stage('Dev deployment') {
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                environment {
                    DOCKER_IMAGE = "jenkins_fastapi_casts"
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
                    helm upgrade --install app fastapi --values=values.yml --namespace dev
                    '''
                }
            }
            steps {
                environment {
                    DOCKER_IMAGE = "jenkins_fastapi_movies"
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
                environment {
                    DOCKER_IMAGE = "jenkins_fastapi_casts"
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
                    helm upgrade --install app fastapi --values=values.yml --namespace qa
                    '''
                }
            }
            steps {
                environment {
                    DOCKER_IMAGE = "jenkins_fastapi_movies"
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
                environment {
                    DOCKER_IMAGE = "jenkins_fastapi_casts"
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
                    helm upgrade --install app fastapi --values=values.yml --namespace staging
                    '''
                }
            }
            steps {
                environment {
                    DOCKER_IMAGE = "jenkins_fastapi_movies"
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
                environment {
                    DOCKER_IMAGE = "jenkins_fastapi_casts"
                }
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
            steps {
                environment {
                    DOCKER_IMAGE = "jenkins_fastapi_movies"
                }
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
