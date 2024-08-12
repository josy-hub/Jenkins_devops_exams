pipeline {
environment {
DOCKER_ID = "josydocker22"
DOCKER_REPO = "jenkins_devops_exams"
DOCKER_IMAGE_CAST = "jenkins_devops_exams_cast_service"
DOCKER_IMAGE_MOVIE = "jenkins_devops_exams_movie_service"
DOCKER_TAG = "v.${BUILD_ID}.0"
BRANCH_NAME = "${env.GIT_BRANCH}"
}
agent any
stages {
        stage(' Docker Build'){
            steps {
                script {
                sh '''
                 docker-compose build
                 APPS="cast movie"
                 for APP in $APPS; do
                 docker tag "jenkins_devops_exams_${APP}_service:latest" "$DOCKER_ID/$DOCKER_REPO:${APP}_$DOCKER_TAG"; done
                sleep 10
                '''
                }
            }
        }
        stage('Docker run'){
                steps {
                    script {
                    sh '''
                    docker-compose up -d
                    docker ps
                    sleep 10
                    '''
                    }
                }
            }

        stage('Test Acceptance'){
            steps {
                    script {
                    echo "BRANCH_NAME: ${BRANCH_NAME}"
                    sh '''
                    curl localhost:8001/api/v1/casts/docs
                    curl localhost:8002/api/v1/movies/docs
                    '''
                    }
            }

        }
        stage('Docker Push'){
            environment
            {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS")
            }

            steps {

                script {
                sh '''
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
        KUBECONFIG = credentials("config")
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
                helm upgrade --install moviecast-api moviecast_api_helm --values=values.yml --namespace dev
                '''
                }
            }

        }
stage('Deploiement en qa'){
        environment
        {
        KUBECONFIG = credentials("config")
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
                helm upgrade --install moviecast-api moviecast_api_helm --values=values.yml --namespace qa
                '''
                }
            }

        }
stage('Deploiement en staging'){
        environment
        {
        KUBECONFIG = credentials("config")
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
                helm upgrade --install moviecast-api moviecast_api_helm --values=values.yml --namespace staging
                '''
                }
            }

        }
stage('Deploiement en prod'){
        environment
        {
          KUBECONFIG = credentials("config")
        }
        when {
          expression {
            return BRANCH_NAME ==~ /(origin\/main|origin\/master)/
          }
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
                cp moviecast_api_helm/values.yaml values.yml
                cat values.yml
                helm upgrade --install moviecast-api moviecast_api_helm --values=values.yml --namespace prod
                '''
                }
            }

        }

}
}
