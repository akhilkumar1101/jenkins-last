pipeline {

    agent any

    tools {
        jdk 'JDK21'
        maven 'maven-3.9'
    }

    environment {
        IMAGE = 'ghcr.io/akhilkumar1101/jenkins-last'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn package -DskipTests'
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    docker build \
                    -t ${IMAGE}:${BUILD_NUMBER} \
                    -t ${IMAGE}:latest .
                '''
            }
        }

        stage('Push to GHCR') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'github-packages-cred',
                        usernameVariable: 'USERNAME',
                        passwordVariable: 'TOKEN'
                    )
                ]) {
                    sh '''
                        echo "$TOKEN" | docker login ghcr.io \
                            -u "$USERNAME" \
                            --password-stdin

                        docker push ${IMAGE}:${BUILD_NUMBER}
                        docker push ${IMAGE}:latest

                        docker logout ghcr.io
                    '''
                }
            }
        }
    }

    post {

        success {
            emailext(
                subject: "SUCCESS: ${JOB_NAME} #${BUILD_NUMBER}",
                body: "Pipeline completed successfully.\n\nDocker Image: ${IMAGE}:${BUILD_NUMBER}",
                to: "YOUR_EMAIL@example.com"
            )
        }

        failure {
            emailext(
                subject: "FAILED: ${JOB_NAME} #${BUILD_NUMBER}",
                body: "Pipeline failed.\n\nCheck Jenkins console output:\n${BUILD_URL}",
                to: "YOUR_EMAIL@example.com"
            )
        }
    }
}
