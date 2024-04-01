#!/usr/bin/env groovy

import groovy.transform.Field

@Field
String DOCKER_USER_REF = 'd-dockerhub'
@Field
String SSH_ID_REF = 'ssh-credentials-id'

pipeline {
    agent any

    tools {
        dockerTool 'docker'
    }

    stages {
        stage("build and test") {
            steps {
                sh "ls -la"
                sh "docker build -t danhdz1310/my-repo:latest ."
            }
        }
        stage("Docker login and push docker image") {
            steps {
                withBuildConfiguration {
                    sh 'docker login --username ${repository_username} --password ${repository_password}'
                    sh "docker push danhdz1310/my-repo:latest"
                }
            }
        }
        stage("deploy") {
            steps {
                withBuildConfiguration {
                    sshagent(credentials: [SSH_ID_REF]) {
                        sh '''
                            ssh -o StrictHostKeyChecking=no root@ec2-18-143-167-76.ap-southeast-1.compute.amazonaws.com "docker run --detach --name danh-todo-app -p 9090:8000 danhdz1310/my-repo:latest"
                            docker ps
                        '''
                    }
                }
            }
        }
    }
}

void withBuildConfiguration(Closure body) {
    withCredentials([usernamePassword(credentialsId: DOCKER_USER_REF, usernameVariable: 'repository_username', passwordVariable: 'repository_password')]) {
        body()
    }
}
