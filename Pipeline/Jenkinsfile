pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                script{
                    dockerbackend = docker.build("tcc/api:${env.BUILD_ID}", '-f ./Back/Dockerfile ./Back')
                    dockerFrontend = docker.build("tcc/spa:${env.BUILD_ID}", '-f ./Front/Dockerfile ./Front')
                }
            }
        }
        stage('Push Docker Images'){
            steps{
                script {
                    docker.withRegistry("https://harbor.festivalculturalptu.shop:8443", "harbor") {
                        dockerbackend.push('latest')
                        dockerFrontend.push('latest')
                        dockerbackend.push("${env.BUILD_ID}")
                        dockerFrontend.push("${env.BUILD_ID}")
                    }
                }
            }
        }
        stage('Deploy') {
            steps {
                withCredentials(
                [usernamePassword(credentialsId: 'harbor', usernameVariable: 'HARBOR_USER', passwordVariable: 'HARBOR_PASS'), 
                string(credentialsId: 'server-ip', variable: 'SSH_HOST')]) {
                    sshagent(['id-servidor']) {
                        sh '''
                            ssh -o StrictHostKeyChecking=no augusto@${SSH_HOST} '
                            cd /home/augusto/Adesp

                            echo "$HARBOR_PASS" | docker login harbor.festivalculturalptu.shop:8443 -u "$HARBOR_USER" --password-stdin

                            docker compose pull
                            docker compose up -d
                            ' 
                        '''
                    }
                }
            }
        }
    }
}