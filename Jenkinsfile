pipeline {
    agent any

    tools {
        jdk 'jdk'
        maven 'maven'
        // docker 'docker'
        // node 'nodejs'
    }

    environment {
        SCANNER_HOME = tool 'sonar-tools'
        DOCKER_REGISTRY = 'docker.io'
        DOCKER_REPO = 'aswines'
        DOCKER_IMAGE = 'java'
        DOCKER_TAG = "${env.BUILD_NUMBER}"
    }

    stages {
        stage('Clean Workspace') {
            steps {
                deleteDir()  // This will clean the workspace before starting the build
            }
        }

        stage('Git Checkout') {
            steps {
                git branch: 'main', changelog: false, poll: false, url: 'https://github.com/AswinES97/Java-Devops.git'
            }
        }

        stage('Compile') {
            steps {
                sh 'mvn clean install -Dmaven.test.skip=true'
            }
        }

        stage('Sonar') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner \
                    -D sonar.projectName=cicd \
                    -D sonar.java.binaries=. \
                    -D sonar.projectKey=cicd '''
                }
            }
        }

        stage('Login to Docker Registry') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'dockerHubToken', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                        sh '''
                            echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin $DOCKER_REGISTRY
                        '''
                    }
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    sh '''
                        docker build -t $DOCKER_REPO/$DOCKER_IMAGE:$DOCKER_TAG .
                        docker push $DOCKER_REPO/$DOCKER_IMAGE:$DOCKER_TAG
                    '''
                }
            }
        }
    }

    post {
        always {
            // Ensure Docker logout is performed
            script {
                sh '''
                    docker logout $DOCKER_REGISTRY
                '''
            }
        }
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}
