// AJOUT DE DOCKER RUN NGINX, DB CASTS, DB MOVIES
pipeline {
    agent any
    environment {
        DOCKER_ID = "p0l1na"
        DOCKER_TAG = "v.${BUILD_ID}.0"
        DOCKER_PASS = credentials("DOCKER_HUB_PASS")
        DOCKER_CAST_IMAGE = "cast-service"
        DOCKER_MOVIE_IMAGE = "movie-service"
    }
    stages {
        stage('Docker Build') {
            steps {
                script {
                    sh '''
                    if [ $( docker ps -a | grep cast-service movie-service movie-db cast-db nginx | wc -l ) -gt 0 ]; then
                        docker rm -f cast-service movie-service movie-db cast-db nginx
                    fi
                    docker build -t $DOCKER_ID/$DOCKER_CAST_IMAGE:$DOCKER_TAG .
                    docker build -t $DOCKER_ID/$DOCKER_MOVIE_IMAGE:$DOCKER_TAG .
                    sleep 6
                    '''
                }
            }
        }
        stage('Docker run') {
            steps {
                script {
                    sh '''
                    docker run -d --name cast-db /
                        -v postgres_data_cast:/var/lib/postgresql/data/ /
                        -e POSTGRES_USER=cast_db_user /
                        -e POSTGRES_PASSWORD=cast_db_password /
                        -e POSTGRES_DB=cast_db /
                        postgres:12.1-alpine
                    docker run -d --name movie-db /
                        -v postgres_data_movie:/var/lib/postgresql/data/ /
                        -e POSTGRES_USER=movie_db_user /
                        -e POSTGRES_PASSWORD=movie_db_password /
                        -e POSTGRES_DB=movie_db /
                        postgres:12.1-alpine
                    docker run -d --name cast-service
                        -p 8002:8000 /
                        -e DATABASE_URI=postgresql://cast_db_user:cast_db_password@cast-db:5432/cast_db /
                        --link cast-db /
                        $DOCKER_ID/$DOCKER_CAST_IMAGE:$DOCKER_TAG
                    docker run -d --name movie-service
                        -p 8001:8000 /
                        -e DATABASE_URI=postgresql://movie_db_user:movie_db_password@movie-db:5432/movie_db /
                        -e CAST_SERVICE_HOST_URL=http://cast_service:8000/api/v1/casts/
                        --link movie-db /
                        $DOCKER_ID/$DOCKER_MOVIE_IMAGE:$DOCKER_TAG
                    docker run -d --name nginx
                        -p 80:8080 /
                        -v /home/ubuntu/nginx.conf:/etc/nginx/nginx.conf /
                        --link movie-service /
                        --link cast-service /
                        nginx:1.17.6-alpine
                    sleep 10
                    '''
                }
            }
        }
        stage('Test Acceptance') {
            steps {
                script {
                    sh '''
                    curl localhost
                    '''
                }
            }
        }
        stage('Docker Push') {
            steps {
                script {
                    sh '''
                    docker login -u $DOCKER_ID -p $DOCKER_PASS
                    docker push $DOCKER_ID/$DOCKER_MOVIE_IMAGE:$DOCKER_TAG
                    docker push $DOCKER_ID/$DOCKER_CAST_IMAGE:$DOCKER_TAG
                    '''
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
                    cat $KUBECONFIG > .kube/config
                    helm -n dev upgrade --install movie-db --values=movie-db-helm/values.yml
                    helm -n dev upgrade --install cast-db --values=cast-db-helm/values.yml
                    helm -n dev upgrade --install movie-service --values=movie-helm/values.yml
                    helm -n dev upgrade --install cast-service --values=cast-helm/values.yml
                    helm -n dev upgrade --install nginx --values=nginx-helm/values.yml
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
                    cat $KUBECONFIG > .kube/config
                    helm -n qa upgrade --install movie-db --values=movie-db-helm/values.yml
                    helm -n qa upgrade --install cast-db --values=cast-db-helm/values.yml
                    helm -n qa upgrade --install movie-service --values=movie-helm/values.yml
                    helm -n qa upgrade --install cast-service --values=cast-helm/values.yml
                    helm -n qa upgrade --install nginx --values=nginx-helm/values.yml
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
                    cat $KUBECONFIG > .kube/config
                    helm -n staging upgrade --install movie-db --values=movie-db-helm/values.yml
                    helm -n staging upgrade --install cast-db --values=cast-db-helm/values.yml
                    helm -n staging upgrade --install movie-service --values=movie-helm/values.yml
                    helm -n staging upgrade --install cast-service --values=cast-helm/values.yml
                    helm -n staging upgrade --install nginx --values=nginx-helm/values.yml
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
                    cat $KUBECONFIG > .kube/config
                    helm -n prod upgrade --install movie-db --values=movie-db-helm/values.yml
                    helm -n prod upgrade --install cast-db --values=cast-db-helm/values.yml
                    helm -n prod upgrade --install movie-service --values=movie-helm/values.yml
                    helm -n prod upgrade --install cast-service --values=cast-helm/values.yml
                    helm -n prod upgrade --install nginx --values=nginx-helm/values.yml
                    '''
                }
            }
        }
    }
}
