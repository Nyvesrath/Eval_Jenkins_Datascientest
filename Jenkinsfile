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

        stage('Docker run'){ // run container from our built image
            steps {
                script {
                    sh '''
                    docker run -d -p 8000:8000 --name jenkins_cast $DOCKER_ID/$DOCKER_IMAGE_CAST:$DOCKER_TAG
                    sleep 3
                    docker run -d -p 8000:8000 --name jenkins_movie $DOCKER_ID/$DOCKER_IMAGE_MOVIE:$DOCKER_TAG
                    sleep 3
                    '''
                }
            }
        }

        stage('Test Acceptance'){ // we launch the curl command to validate that the container responds to the request
            steps {
                script {
                    sh '''
                    curl  http://localhost:8080/api/v1/casts/docs 
                    curl  http://localhost:8080/api/v1/movies/docs 
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
                    cp charts/values.yaml values.yml
                    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                    helm upgrade --install app charts --values=values.yml --namespace dev
                    '''
                }
            }
        }
    }
}