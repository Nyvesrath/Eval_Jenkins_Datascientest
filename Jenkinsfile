pipeline {
    environment { // Declaration of environment variables
        DOCKER_ID = "nyvesrath" // replace this with your docker-id
        DOCKER_IMAGE_CAST = "evaljenkinscast"
        DOCKER_IMAGE_MOVIE = "evaljenkinsmovie"
        DOCKER_TAG = "v.${BUILD_ID}.0" // we will tag our images with the current build in order to increment the value by 1 with each new build
    }

    agent any // Jenkins will be able to select all available agents
    stages {
        stage('Docker Build'){ // docker build image stage
            steps {
                script {
                    sh '''
                        docker rm -f jenkins_cast
                        docker rm -f jenkins_movie

                        docker build -t $DOCKER_ID/$DOCKER_IMAGE_CAST:$DOCKER_TAG ./cast-service/
                        sleep 3
                        docker build -t $DOCKER_ID/$DOCKER_IMAGE_MOVIE:$DOCKER_TAG ./movie-service/
                        sleep 3
                    '''
                }
            }
        }

        stage('Docker Push'){ //we pass the built image to our docker hub account
            environment {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS") // we retrieve  docker password from secret text called docker_hub_pass saved on jenkins
            }
            steps {
                script {
                    sh '''
                    docker login -u $DOCKER_ID -p $DOCKER_PASS
                    docker push $DOCKER_ID/$DOCKER_IMAGE_CAST:$DOCKER_TAG
                    docker push $DOCKER_ID/$DOCKER_IMAGE_MOVIE:$DOCKER_TAG
                    '''
                }
            }
        }

        //----------------------------------- Deploiement -----------------------------------
        stage('Deploiement en dev'){
            environment {
                KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
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
    }
}