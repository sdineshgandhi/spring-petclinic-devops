pipeline {
    agent { label 'ubuntu-wsl' }

    environment {
        IMAGE_NAME = 'sdineshgandhi/spring-petclinic'
        IMAGE_TAG  = "${BUILD_NUMBER}"

        OCTOPUS_SPACE   = 'Default'
        OCTOPUS_PROJECT = 'Spring Petclinic'
    }

    stages {

        stage('Checkout') {
            steps {
                git(
                    credentialsId: 'github-cred',
                    url: 'https://github.com/sdineshgandhi/spring-petclinic-devops.git',
                    branch: 'main'
                )
            }
        }

        stage('Maven Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    docker build \
                        -t ${IMAGE_NAME}:${IMAGE_TAG} \
                        .
                '''
            }
        }

        stage('Trivy Scan') {
            steps {
                sh '''
                    trivy image ${IMAGE_NAME}:${IMAGE_TAG}
                '''
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-cred',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_PASSWORD" | docker login \
                            -u "$DOCKER_USER" \
                            --password-stdin

                        docker push ${IMAGE_NAME}:${IMAGE_TAG}

                        docker logout
                    '''
                }
            }
        }

        stage('Create Octopus Release') {
            steps {
                withCredentials([
                    string(
                        credentialsId: 'octopus-api-key',
                        variable: 'OCTOPUS_API_KEY'
                    ),
                    string(
                        credentialsId: 'octopus-url',
                        variable: 'OCTOPUS_URL'
                    )
                ]) {
                    sh '''
                        octopus release create \
                            --server "$OCTOPUS_URL" \
                            --apiKey "$OCTOPUS_API_KEY" \
                            --space "$OCTOPUS_SPACE" \
                            --project "$OCTOPUS_PROJECT" \
                            --releaseNumber "$BUILD_NUMBER"
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "Petclinic CI/CD pipeline completed successfully."
            echo "Docker image: ${IMAGE_NAME}:${IMAGE_TAG}"
            echo "Octopus release: ${BUILD_NUMBER}"
        }

        failure {
            echo "Pipeline failed. Check the failed stage in the Jenkins console."
        }
    }
}
