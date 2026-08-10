pipeline {
    agent {
        label 'java'
    }

    options {
        timeout(time: 30, unit: 'MINUTES')

        buildDiscarder(
            logRotator(
                numToKeepStr: '20'
            )
        )

        disableConcurrentBuilds()
    }

    environment {
        APP_NAME = 'jenkins-springboot'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean compile'
            }
        }

        stage('Unit Tests') {
            steps {
                sh 'mvn test'
            }

            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }

        stage('Package') {
            steps {
                sh 'mvn package -DskipTests'

                archiveArtifacts(
                                artifacts: 'target/*.jar',
                                fingerprint: true
                            )
            }
        }
    }

    post {
        success {
            echo "${APP_NAME} CI succeeded."
        }

        failure {
            echo "${APP_NAME} CI failed."
        }

        always {
            echo 'Pipeline execution completed.'
        }
    }
}