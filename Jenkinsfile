#!/bin/groovy

def dockerImg

pipeline {
    agent any
    stages {
        stage ('Clone Repo') {
            steps {
                /* Clone git Repository */
                checkout scm
            }
        }

        stage ('Test Package') {
            steps {
                /* test application using maven */
                sh 'mvn test'
            }
            post {
                always {
                    /* Perform JUnit testing
                    * sh './gradlew check' */
                    archiveArtifacts "target/**/*"
                    junit '**/target/*-reports/TEST-*.xml'
                }
            }
        }

        stage ('Compile & Package') {
            steps {
                /* Package Application using maven */
                sh 'mvn package'
            }
        }

        stage ('SonaQube Analysis') {
            steps {
                /* Static code analysis with SonarQube */
                withSonarQubeEnv('sonar') {
                    sh 'mvn sonar:sonar'
                }
            }
        }

        stage ('Build Image') {
            steps {
                echo "Build docker image"
                script {
                    /* This builds the actual image; synonymous to
                    * docker build on the command line */
                    dockerImg = docker.build("juned/myapp")
                }
            }
        }

        stage ('Push Image') {
            steps {
                echo "Push Image on DockerHub"
                script {
                    /* Finally, we'll push the image with two tags:
                    * First, the incremental build number from Jenkins
                    * Second, the 'latest' tag. */
                    docker.withRegistry('https://registry.hub.docker.com', 'docker-hub-credentials') {
                        dockerImg.push("${env.BUILD_NUMBER}")
                        dockerImg.push("latest")
                    }
                }
            }
        }
    }

    post {
        success {
            echo "Send success email notification"
            mail bcc: '', body: " Build Status : Success <br>Project: ${env.JOB_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br>Build URL: ${env.BUILD_URL} <br>App URL: http://34.93.78.26/addressbook", cc: '', charset: 'UTF-8',  mimeType: 'text/html', replyTo: '', subject: "Jenkins Build Status -> ${env.JOB_NAME}", to: "jkhan6722@gmail.com";
        }
        failure {
            echo "send failure email notification"
            mail bcc: '', body: " Build Status : Failed <br>Project: ${env.JOB_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br>Build URL: ${env.BUILD_URL}", cc: '', charset: 'UTF-8',  mimeType: 'text/html', replyTo: '', subject: "Jenkins Build Status -> ${env.JOB_NAME}", to: "jkhan6722@gmail.com";
        }
    }
}
