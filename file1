pipeline {
    agent any

    environment {
        MAVEN_OPTS = '-Dmaven.repo.local=.m2/repository'
        APP_PORT = '8080'                // Example environment variable for application
        IMAGE_NAME = 'spring-boot-app'   // Docker image name
    }

    options {
        timestamps()                     // Adds timestamps to logs
        skipStagesAfterUnstable()        // Stops pipeline execution after an unstable stage
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out the code...'
                checkout scm
            }
        }

        stage('Build') {
            steps {
                echo 'Building the application...'
                sh './mvnw clean package -DskipTests' // Skip tests in the build stage
            }
        }

        stage('Test') {
            parallel {
                stage('Unit Tests') {
                    steps {
                        echo 'Running unit tests...'
                        sh './mvnw test'
                    }
                }
                stage('Integration Tests') {
                    steps {
                        echo 'Running integration tests...'
                        sh './mvnw verify -Pintegration-tests'
                    }
                }
            }
        }

        stage('Docker Build') {
            steps {
                echo 'Building Docker image...'
                sh """
                    docker build -t ${IMAGE_NAME}:latest .
                """
            }
        }

        stage('Run in Docker') {
            steps {
                echo 'Running the application in Docker...'
                sh """
                    docker run -d --name spring-boot-container -p ${APP_PORT}:${APP_PORT} ${IMAGE_NAME}:latest
                """
            }
        }

        stage('Archive Artifacts') {
            steps {
                echo 'Archiving build artifacts...'
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        }
    }

    post {
        success {
            echo 'Build, tests, and deployment were successful!'
            slackSend(channel: '#builds', message: "SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER} completed successfully!")
        }
        failure {
            echo 'There was a problem with the build, tests, or deployment.'
            slackSend(channel: '#builds', message: "FAILURE: ${env.JOB_NAME} #${env.BUILD_NUMBER} failed. Check logs.")
        }
    }
}
