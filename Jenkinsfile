pipeline{

    agent any

    environment {
        BRANCH_NAME = "${GIT_BRANCH.split("/")[1]}"
    }

    stages{
        stage('Echo') {
            steps{
                sh "echo Je suis là"
                }
            }
        stage('Show Release') {
            steps {
                sh "cat /etc/os-release"
            }
        }
        stage('Initialisation') {
            steps {
                sh "make clean"
                sh "make venv" 
                sh "make install"
            }
        }

        stage('Unit Test') {
            steps {
                sh "make test"
            }
        }

        stage('Build docker image') {
            steps {
                sh "docker ps -a" 
                sh "echo $BRANCH_NAME"
                sh "echo $env.GIT_BRANCH"
                sh "printenv"
                sh "docker build -t hervlokossou/$env.BRANCH_NAME/default_image_$env.BRANCH_NAME ."
            }
        }


        stage('Test docker image') { 
            steps {
                sh "docker run -d -p 5001:8000 --name default_container_$env.BRANCH_NAME hervlokossou/$env.BRANCH_NAME/default_image_$env.BRANCH_NAME"
            }
        }

                
        stage('Cleanup ') {
            steps {
                sh "docker container stop default_container_$env.BRANCH_NAME"
                sh "docker container rm default_container_$env.BRANCH_NAME" 
            }
        }

        stage("Push image to Docker Hub"){
            steps{
                withCredentials([usernamePassword(credentialsId: 'DOCKER_CREDS', passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]){
                    sh """
                    docker login  --username $USERNAME --password $PASSWORD && \
                    docker push hervlokossou/${BRANCH_NAME}/default_image_$env.BRANCH_NAME:latest
                    """
                }
            }
        }
    } 

    post{
        always {
            sh "docker container stop default_container_$env.BRANCH_NAME"
            sh "docker container rm default_container_$env.BRANCH_NAME" 
            }
    }    
}
