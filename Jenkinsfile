pipeline {
    agent any
    tools {
        maven 'maven-3.8.4' 
    }
      environment {
        DATE = new Date().format('yy.M')
        TAG = "${DATE}.${BUILD_NUMBER}"
    }
    stages{
        stage('build') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('Docker Build') {
            steps {
                script {
                    docker.build("spandy1932/aws-rds:${TAG}")
                }
            }
        }
        stage('Pushing Docker Image to Dockerhub') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com','docker_credentials') {
                        docker.image("spandy1932/aws-rds:${TAG}").push()
                        docker.image("spandy1932/aws-rds:${TAG}").push("latest")
                    }
                }
            }
        }
        stage('deploy') {
            steps {
                sshagent(credentials:['sshslave']) {
                    sh 'ssh -T -o StrictHostKeyChecking=no devops@172.31.95.168'
                    sh 'docker stop aws-rds | true'
                    sh 'docker rm aws-rds | true'
                    sh 'docker run --name aws-rds -d -p 9005:8080 spandy1932/aws-rds:${TAG}'
         
                }
            }
        }
    }
}    
