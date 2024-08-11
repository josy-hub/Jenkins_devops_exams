pipeline {
environment { // Declaration of environment variables
DOCKER_ID = "josydocker22" // replace this with your docker-id
DOCKER_REPO = "jenkins_devops_exams"
DOCKER_IMAGE_CAST = "jenkins_devops_exams_cast_service"
DOCKER_IMAGE_MOVIE = "jenkins_devops_exams_movie_service"
DOCKER_TAG = "v.${BUILD_ID}.0" // we will tag our images with the current build in order to increment the value by 1 with each new build
}
agent any // Jenkins will be able to select all available agents
stages {
        stage(' Docker Build'){ // docker build image stage
            steps {
                script {
                sh '''
                 #!/bin/bash
                 docker-compose build
                 APPS="cast movie"
                 for APP in $APPS; do
                 docker tag "jenkins_devops_exams_$APP_service:latest" "$DOCKER_ID/$DOCKER_REPO:${APP}_$DOCKER_TAG"; done
                sleep 10
                '''
                }
            }
        }
        stage('Docker run'){ // run container from our builded image
                steps {
                    script {
                    sh '''
                    docker-compose up -d
                    docker ps
                    docker logs
                    sleep 10
                    '''
                    }
                }
            }

        stage('Test Acceptance'){ // we launch the curl command to validate that the container responds to the request
            steps {
                    script {
                    sh '''
                    curl localhost:8080/
                    '''
                    }
            }

        }
        stage('Docker Push'){ //we pass the built image to our docker hub account
            environment
            {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS") // we retrieve  docker password from secret text called docker_hub_pass saved on jenkins
            }

            steps {

                script {
                sh '''
                #!/bin/bash
                APPS="cast movie"
                docker login -u $DOCKER_ID -p $DOCKER_PASS
                for APP in $APPS; do
                docker push "$DOCKER_ID/$DOCKER_REPO:${APP}_$DOCKER_TAG"; done
                '''
                }
            }

        }

stage('Deploiement en dev'){
        environment
        {
        KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
        }
            steps {
                script {
                sh '''
                rm -Rf .kube
                mkdir .kube
                ls
                cat $KUBECONFIG > .kube/config
                cp moviecast_api_helm/values.yaml values.yml
                cat values.yml
                #sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                helm upgrade --install moviecast-api moviecast_api_helm --values=values.yml --namespace dev
                '''
                }
            }

        }
stage('Deploiement en qa'){
        environment
        {
        KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
        }
            steps {
                script {
                sh '''
                rm -Rf .kube
                mkdir .kube
                ls
                cat $KUBECONFIG > .kube/config
                cp moviecast_api_helm/values.yaml values.yml
                cat values.yml
                #sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                helm upgrade --install moviecast-api moviecast_api_helm --values=values.yml --namespace qa
                '''
                }
            }

        }
stage('Deploiement en staging'){
        environment
        {
        KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
        }
            steps {
                script {
                sh '''
                rm -Rf .kube
                mkdir .kube
                ls
                cat $KUBECONFIG > .kube/config
                cp moviecast_api_helm/values.yaml values.yml
                cat values.yml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                helm upgrade --install moviecast-api moviecast_api_helm --values=values.yml --namespace staging
                '''
                }
            }

        }
stage('Deploiement en prod'){
        environment
        {
        KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
        }
            steps {
               when {
                 expression {
                    return env.BRANCH_NAME == 'main' || env.BRANCH_NAME == 'master'
                 }
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
                cp moviecast_api_helm/values.yaml values.yml
                cat values.yml
                #sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                helm upgrade --install moviecast-api moviecast_api_helm --values=values.yml --namespace prod
                '''
                }
            }

        }

}
}
