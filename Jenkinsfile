pipeline {
    agent any

    environment {
        DOCKER_REGISTRY = 'localhost:5001'
        ARTIFACTORY_REPO = 'gradle-dev-local'
        ARTIFACTORY_DOCKER_REGISTRY = 'docker-local'
        IMAGE_NAME = 'jfrog-test/spring-petclinic'
        IMAGE_TAG = 'latest'
        ARTIFACTORY_CREDENTIALS = credentials('artifactory-credentials-id')
    }

    tools {
        jfrog 'jfrog-cli'
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

        stage('Publish JARs to Artifactory using jFrog Plugin') {
            steps {
                script {
// can use a better way to extract application group
                    def group = 'org.springframework.samples'
                    def version = '3.3.0'

                    def jarFile = findFiles(glob: 'build/libs/*.jar')[1].path
                    def repoPath = "${group.replace('.', '/')}/${version}/${jarFile.split('/')[-1]}"
            
                    echo "Group: ${group}, Version: ${version}, Path: ${repoPath}"

                    jf "rt u $jarFile ${ARTIFACTORY_REPO}/$repoPath"
                }
            }
        }

       stage('Pull Dockerfile from Github') {
            steps {
                script {
                  sh 'curl -o Dockerfile https://raw.githubusercontent.com/poprygun/jenkins-cicd/master/Dockerfile'
                }
            }
        }

        stage('Package Application Docker Image') {
            steps {
                script {
                    sh 'docker build -t ${DOCKER_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG} .'
                }
            }
        }

        stage('Push Docker Image to Artifactory') {
            steps {
                script {
                    // todo switch to use jFrog CLI when Artifactory Edition that supports Docker Registry is available
                    // for now just use mock registry started from docker compose
                    
                    sh 'docker push ${DOCKER_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}'
                    
                    // jf 'docker push ${DOCKER_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG} ${ARTIFACTORY_DOCKER_REGISTRY}'
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
