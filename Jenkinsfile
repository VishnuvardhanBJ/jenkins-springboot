pipeline {
    agent {
        label 'docker'
    }

    options {
        timeout(time: 15, unit: 'MINUTES')

        buildDiscarder(
            logRotator(
                numToKeepStr: '20'
            )
        )

        disableConcurrentBuilds()
    }

    environment {
        APP_NAME = 'jenkins-springboot'
        IMAGE_TAG = "${BUILD_NUMBER}"
        DOCKER_IMAGE = "vishnuvardhanbj/jenkins-springboot:${BUILD_NUMBER}"
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
                    junit '**/target/surefire-reports/*.xml'
                }
            }
        }

//         stage('Security Scans') {
//             parallel {
                stage('SonarQube Analysis') {
                    steps {
                        withSonarQubeEnv('sonarqube') {
                            sh 'mvn sonar:sonar'
                        }
                    }
                }
//
//                 stage('Dependency Scan') {
//                     steps {
//                         sh 'mvn dependency-check:check'
//                     }
//                 }
//
//                 stage('Secret Scan') {
//                     steps {
//                         sh 'gitleaks detect --source . --exit-code 1'
//                     }
//                 }
//             }
//         }

        stage('Quality Gate') {
            steps {
                timeout(time : 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
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

            post {
                success {
                    archiveArtifacts artifacts: '**/target/*.jar, **/target/*.war', fingerprint: true
                }
            }
        }

        stage('Docker Build') {
            steps {
//                 sh "docker build -t jenkins-springboot:${BUILD_NUMBER} ."
                sh "docker build -t ${APP_NAME}:${IMAGE_TAG} -t ${DOCKER_IMAGE} ."
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin

                        docker push "$DOCKER_IMAGE"

                        docker logout
                    '''
                }
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    docker rm -f springboot-app || true

                    docker run -d --name springboot-app -p 8081:8090 jenkins-springboot:${BUILD_NUMBER}

                    sleep 10

                    docker ps --filter "name=springboot-app"

                    docker logs springboot-app
                '''
            }
        }

//
//         stage('Container Security Scan') {
//             steps {
//                 sh "trivy image --severity HIGH,CRITICAL --exit-code 1 jenkins-springboot:${BUILD_NUMBER}"
//             }
//         }
    }

    post {
        success {
            echo "Docker ${APP_NAME}:${IMAGE_TAG} runs successfully."
        }

        failure {
            echo "Pipeline failed."
        }

        always {
            echo 'Pipeline execution completed.'
        }
    }
}