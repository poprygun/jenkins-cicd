pipeline {
    agent any

    environment {
        DOCKER_REGISTRY = 'localhost:5001'
        IMAGE_NAME = 'jfrog-test/spring-petclinic'
        IMAGE_TAG = 'latest'
        ARTIFACTORY_CREDENTIALS = credentials('artifactory-credentials-id')
    }

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/spring-projects/spring-petclinic.git'
            }
        }

        stage('Compile Code') {
            steps {
                sh './gradlew clean build'
            }
        }

        stage('Run Tests') {
            steps {
                sh './gradlew test'
            }
        }
       stage('Create Dockerfile') {
            steps {
                script {
                    writeFile file: 'Dockerfile', text: '''
                        FROM eclipse-temurin:17-jre-alpine

                        WORKDIR /app

                        COPY build/libs/*.jar app.jar

                        EXPOSE 8080

                        ENTRYPOINT ["java", "-jar", "/app/app.jar"]
                    '''
                }
            }
        }
        stage('Package Application') {
            steps {
                script {
                    sh './gradlew build -x test'
                    sh 'docker build -t ${DOCKER_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG} .'
                }
            }
        }

        stage('Push Docker Image to Artifactory') {
            steps {
                script {
                    sh 'docker login -u ${ARTIFACTORY_CREDENTIALS_USR} -p ${ARTIFACTORY_CREDENTIALS_PSW} ${DOCKER_REGISTRY}'
                    sh 'docker push ${DOCKER_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}'
                }
            }
        }
    }

    post {
        always {
            sh 'docker rmi ${DOCKER_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG} || true'
        }
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check logs for errors.'
        }
    }
}
