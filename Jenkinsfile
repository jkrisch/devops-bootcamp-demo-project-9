#!/user/bin/env groovy
library identifier: 'my-shared-lib@main', retriever: modernSCM(
    [
        $class: 'GitSCMSource',
        remote: 'https://github.com/jkrisch/devops-bootcamp-project-8-shared-library.git'
        //in case the repo is private:
        //credentialsId:<<name-of-credentials-in-jenkins-store>>
    ]
)
pipeline {
    agent any

   tools {
        maven 'maven-3.9'
    }
    
   environment{
    IMAGE_NAME = "jaykay84/demo-app:maven-1.0"
    ELASTIC_IP = "18.184.8.248"
   }
    stages {
        stage('build app') {
            steps {
                script {
                    buildJar()
                }
            }
        }

        stage('build image'){
            steps{
                script{
                    withCredentials([
                        usernamePassword(credentialsId:'docker-login', passwordVariable: 'PASS', usernameVariable: 'USER')
                        ]){
                            dockerLogin(USER, PASS)

                            buildImage("jaykay84", "demo-app", "maven-1.0")

                            dockerPush("jaykay84", "demo-app", "maven-1.0")
                        }
                }
            }
        }

        stage('deploy') {
            // input gives the user the oportunity to choose between different parameters in a certain stage
            // for instance if you want to let the developer decide in which environment the build artifact should be deployed to.
            /*input{
                message "Select the environment to deploy to"
                ok "Done"
                parameters{
                    choice(name: 'ENV', choices: ['dev', 'staging', 'prod'], description: '')
                }
            }*/
            steps {
                script {
                    echo 'deploying docker image to EC2...'
                    def dockerCmd = "docker-compose up -d"
                    sshagent(['ec2-server-key']) {
                        sh "scp compose.yaml ec2-user@${ELASTIC_IP}:"
                        sh "ssh -o StrictHostKeyChecking=no ec2-user@${ELASTIC_IP} ${dockerCmd}"
                    }
                }

            }
        }
    }
}
