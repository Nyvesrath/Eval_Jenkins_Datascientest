pipeline {
    environment {
        DOCKER_ID = "nyvesrath" 
        DOCKER_IMAGE_CAST = "evaljenkinscast"
        DOCKER_IMAGE_MOVIE = "evaljenkinsmovie"
        DOCKER_TAG = "v.${BUILD_ID}.0"
    }

    agent any
    stages {
        stage('Docker Build'){ 
            steps {
                script {
                    sh '''
                        docker build -t $DOCKER_ID/$DOCKER_IMAGE_CAST:$DOCKER_TAG ./cast-service/
                        sleep 3
                        docker build -t $DOCKER_ID/$DOCKER_IMAGE_MOVIE:$DOCKER_TAG ./movie-service/
                        sleep 3
                    '''
                }
            }
        }

        stage('Docker Push'){ 
            environment {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS")
            }
            steps {
                script {
                    sh '''
                    docker login -u $DOCKER_ID -p $DOCKER_PASS
                    docker push $DOCKER_ID/$DOCKER_IMAGE_CAST:$DOCKER_TAG
                    docker push $DOCKER_ID/$DOCKER_IMAGE_MOVIE:$DOCKER_TAG

                    sleep 3
                    '''
                }
            }
        }

        //----------------------------------- Deploiement -----------------------------------
        stage('Deploiement en dev'){
            environment {
                KUBECONFIG = credentials("config") 
            }
            steps {
                script {
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    cat $KUBECONFIG > .kube/config
                    helm upgrade --install app-movie-dev ./charts --namespace dev \
                        --set image.repository=$DOCKER_ID/$DOCKER_IMAGE_MOVIE \
                        --set image.tag=$DOCKER_TAG \
                        --set service.nodePort=30081

                    helm upgrade --install app-cast-dev ./charts --namespace dev \
                        --set image.repository=$DOCKER_ID/$DOCKER_IMAGE_CAST \
                        --set image.tag=$DOCKER_TAG \
                        --set service.nodePort=30082
                    '''
                }
            }
        }

        stage('Deploiement en QA'){
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                script {
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    cat $KUBECONFIG > .kube/config
                    helm upgrade --install app-movie-qa ./charts --namespace qa \
                        --set image.repository=$DOCKER_ID/$DOCKER_IMAGE_MOVIE \
                        --set image.tag=$DOCKER_TAG \
                        --set service.nodePort=30083
                        
                    helm upgrade --install app-cast-qa ./charts --namespace qa \
                        --set image.repository=$DOCKER_ID/$DOCKER_IMAGE_CAST \
                        --set image.tag=$DOCKER_TAG \
                        --set service.nodePort=30084
                    '''
                }
            }
        }

        stage('Deploiement en staging'){
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                script {
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    cat $KUBECONFIG > .kube/config
                    helm upgrade --install app-movie-staging ./charts --namespace staging \
                        --set image.repository=$DOCKER_ID/$DOCKER_IMAGE_MOVIE \
                        --set image.tag=$DOCKER_TAG \
                        --set service.nodePort=30085
                        
                    helm upgrade --install app-cast-staging ./charts --namespace staging \
                        --set image.repository=$DOCKER_ID/$DOCKER_IMAGE_CAST \
                        --set image.tag=$DOCKER_TAG \
                        --set service.nodePort=30086
                    '''
                }
            }
        }

        stage('Deploiement en production'){
            when {
                branch 'master'
            }
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
                    helm upgrade --install app-movie-prod ./charts --namespace prod \
                        --set image.repository=$DOCKER_ID/$DOCKER_IMAGE_MOVIE \
                        --set image.tag=$DOCKER_TAG \
                        --set service.nodePort=30087
                        
                    helm upgrade --install app-cast-prod ./charts --namespace prod \
                        --set image.repository=$DOCKER_ID/$DOCKER_IMAGE_CAST \
                        --set image.tag=$DOCKER_TAG \
                        --set service.nodePort=30088
                    '''
                }
            }
        }
    }
}