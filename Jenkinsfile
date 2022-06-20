pipeline {
    
    agent none

    stages {
        stage('worker build') {
            agent {
                docker {
                    image 'maven:3.6.1-jdk-8-alpine'
                }
            }
            when {
                changeset "**/worker/**"
            }
            steps {
                echo 'Compiling worker app'
                dir('worker'){
                    sh 'mvn compile'
                }
                
            }
        }
        stage('worker test') {
            agent {
                docker {
                    image 'maven:3.6.1-jdk-8-alpine'
                }
            }

            when {
                changeset "**/worker/**"
            }
            steps {
                echo 'Testing worker app'
                dir('worker'){
                    sh 'mvn clean test'
                }
            }
        }
        stage('worker docker-package') {
            agent any
            when {
                branch "master"
                changeset "**/worker/**"
            }
            steps {
                echo 'Packaging worker app using docker'
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin') {
                        def workerImage = docker.build("mydock21/worker:v${env.BUILD_ID}", "./worker")
                        workerImage.push("${env.BRANCH_NAME}")
                    }
                }
            }
        }


        stage('result build') {
            when {
                changeset "**/result/**"
            }
            steps {
                echo 'Compiling result app'
                dir('result'){
                    sh 'npm install'
                }
                
            }
        }
        stage('result test') {
            when {
                changeset "**/result/**"
            }
            steps {
                echo 'Testing result app '
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin') {
                        def workerImage = docker.build("mydock21/worker:v${env.BUILD_ID}", "./result")
                        workerImage.push("${env.BRANCH_NAME}")
                    }
                }
            }
        }
        stage('result docker-package') {
            when {
                changeset "**/result/**"
            }
            steps {
                echo 'Packaging result app using docker'
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin') {
                        def workerImage = docker.build("mydock21/worker:v${env.BUILD_ID}", "./result")
                        workerImage.push("${env.BRANCH_NAME}")
                    }
                }
            }
        }


        stage('vote build') {
            agent {
                docker {
                    image 'python:2.7.16-slim'
                    args '--user root'
                }
            }
            when {
                changeset "**/vote/**"
            }
            steps {
                echo 'Compiling vote app'
                dir('vote'){
                    sh 'pip install -r requirements.txt'
                }
                
            }
        }
        stage('vote test') {
            agent {
                docker {
                    image 'python:2.7.16-slim'
                    args '--user root'
                }
            }
            when {
                changeset "**/vote/**"
            }
            steps {
                echo 'Testing vote app'
                dir('vote'){
                    sh 'pip install -r requirements.txt'
                    sh 'nosetests -v'
                }
            }
        }

        stage("vote docker-package") {
            agent any
            when {
                changeset "**/vote/**"
                branch "master"
            }
            steps {
                echo "Packaging vote app with docker"
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin') {
                        def voteImage = docker.build("mydock21/vote:v${env.BUILD_ID}", "./vote")
                        voteImage.push("${env.BRANCH_NAME}")
                    }
                }
            }
        }
    }
    post {
        always {
            echo "Pipeline for mono application is completed"
        }
    }
}
