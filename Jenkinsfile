pipeline {
    agent any

    tools {
        jdk 'java-17'
        maven 'maven'
    }

    environment {
        IMAGE_NAME = "apoorvar12/spring-boot"
        IMAGE_TAG = "v${BUILD_NUMBER}"
    }

    stages {

        stage('Git Clone') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/apoorvaramesh11/java-springboot-application.git'
            }
        }

        stage('Maven Compile') {
            steps {
                sh 'mvn compile'
            }
        }

        stage('Maven Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo '🏗️ Building Docker image...'
                sh 'docker build -t $IMAGE_NAME:$IMAGE_TAG .'
            }
        }

        stage('Docker Login & Push') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: '30f33663-88c0-44b1-8484-e5f5007c6856',
                        passwordVariable: 'DOCKER_PASSWORD',
                        usernameVariable: 'DOCKER_USER'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push $IMAGE_NAME:$IMAGE_TAG
                    '''
                }
            }
        }

        stage('Update K8S manifest & push to Repo') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: '83d36b85-50cd-4f36-88da-5c2aaddfe24b',
                        passwordVariable: 'GIT_PASSWORD',
                        usernameVariable: 'GIT_USERNAME'
                    )
                ]) {
                    sh '''
                        git config user.name "Jenkins"
                        git config user.email "jenkins@example.com"

                        # Pull latest changes first
                        git pull --rebase origin main

                        # Update Kubernetes image
                        sed -i "s|image: .*|image: apoorvar12/spring-boot:${IMAGE_TAG}|" canary/Deployment-2.yaml

                        # Add changed file
                        git add canary/Deployment-2.yaml

                        # Commit
                        git commit -m "Updated Deployment.yaml with build ${IMAGE_TAG}" || echo "No changes to commit"

                        # Push
                        git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/apoorvaramesh11/java-springboot-application.git HEAD:main
                    '''
                }
            }
        }
    }
}
