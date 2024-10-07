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
    IMAGE_NAME = "jaykay84/demo-app"
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

        stage('increment version'){
            steps{
                script{
                    echo 'incrementing app version...'
                    sh 'mvn build-helper:parse-version versions:set \
                    -DnewVersion=\\\${parsedVersion.majorVersion}.\\\${parsedVersion.minorVersion}.\\\${parsedVersion.nextIncrementalVersion} \
                    versions:commit'

                    //read the new version from the pom.xml
                    def matcher = readFile('pom.xml') =~ '<version>(.+?)</version>'
                    def version = matcher[0][1]
                    env.TAG = "$version-$BUILD_NUMBER"
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

                            buildImage("jaykay84", "demo-app", "${env.TAG}")

                            dockerPush("jaykay84", "demo-app", "${env.TAG}")
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
                    def shellCommand = "bash ./server-cmds.sh ${IMAGE_NAME}:${env.TAG}"
                    sshagent(['ec2-server-key']) {
                        sh "scp server-cmds.sh ec2-user@${ELASTIC_IP}:"
                        sh "scp compose.yaml ec2-user@${ELASTIC_IP}:"
                        sh "ssh -o StrictHostKeyChecking=no ec2-user@${ELASTIC_IP} ${shellCommand}"
                    }
                }

            }
        }

        stage('commit version update'){
            steps{
                script{
                    withCredentials([
                        usernamePassword(credentialsId:'github-login', passwordVariable: 'PASS', usernameVariable: 'USER')
                ]){
                    sh '''
                        git config user.email jenkins@example.com
                        git config user.name jenkins-automation
                        git status
                        git branch
                        git config --list
                    '''
                    
                    sh "git remote set-url origin https://${USER}:${PASS}@github.com/jkrisch//devops-bootcamp-demo-project-9.git"
                    
                    sh 'git add pom.xml'
                    sh 'git commit -m "ci: version bump"'
                    sh 'git push origin HEAD:increment-version'
                }
                }
            }
        }
    }
}
