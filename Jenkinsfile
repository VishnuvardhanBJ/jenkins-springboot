pipeline {
    agent {
        label 'docker'
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
//         IMAGE_TAG = "${BUILD_NUMBER}"
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

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh 'mvn sonar:sonar'
                }
            }
        }

        stage('Quality Gate') {
            steps {
                waitForQualityGate abortPipeline: true
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

//         stage('Docker Build') {
//             steps {
//                 sh "docker build -t jenkins-springboot:${BUILD_NUMBER} ."
//             }
//         }
    }

    post {
        success {
//             echo "Docker image ${APP_NAME}:${IMAGE_TAG} built successfully."
               echo "Pipeline built successfully"
        }

        failure {
            echo "Pipeline failed."
        }

        always {
            echo 'Pipeline execution completed.'
        }
    }
}