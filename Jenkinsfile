pipeline {
    agent any
    environment{ 
        dockerImage = "ravikharel/nginx"
    }
    stages {
        stage('Building the docker image'){ 
            steps{ 
                sh "docker image build -t ${dockerImage}:${BUILD_NUMBER} . "
            }
        }
        stage('Scanning the image'){ 
            steps{ 
                sh 'trivy image --exit-code 1 --severity HIGH,CRITICAL --ignore-unfixed $dockerImage:$BUILD_NUMBER'
            }
        }
        stage('Pushing the docker image to the docker hub'){ 
            steps{ 
                withDockerRegistry([credentialsId: 'docker-hub' , url: '']){

                sh '''
                docker push ${dockerImage}:${BUILD_NUMBER}
                '''
                }
                
            }
        }
        stage('Pulling the image to the ansible node'){ 
            agent{ 
                label "ansible-node"

            }
            steps{ 
                sh "docker pull ${dockerImage}:${BUILD_NUMBER}"
            }
        }
        stage("saving it as a tar file"){
            agent{ 
                label "ansible-node"
            }
            steps{ 
                sh " docker save -o $WORKSPACE/nginx:latest.tar ${dockerImage}:${BUILD_NUMBER}"
            }
        }
        stage('Playing the playbook'){ 
            agent{ 
                label "ansible-node"
            }
            steps{ 
                sh '''
                    cd $WORKSPACE
                    ansible-playbook playbook.yml -e "workspace=$WORKSPACE" -e "number=$BUILD_NUMBER"
                '''
            }
        }
    }
    post {
        always { 
            mail to: 'kharel1248@gmail.com',
            subject: "Job '${JOB_NAME}' (${BUILD_NUMBER}) status",
            body: "Please go to ${BUILD_URL} and verify the build"
        }
        success {
            emailext(
                to: 'kharel1248@gmail.com',
                subject: "Jenkins Build ${JOB_NAME} #${BUILD_NUMBER} Success",
                body: """The build was successful.
                Check the job at ${BUILD_URL}""",
            
            )
        }
        failure {
            emailext(
                to: 'kharel1248@gmail.com',
                subject: "Jenkins Build ${JOB_NAME} #${BUILD_NUMBER} Failed",
                body: """The build failed. Please check the logs at ${BUILD_URL}""",
                
            )
        }
    }
}