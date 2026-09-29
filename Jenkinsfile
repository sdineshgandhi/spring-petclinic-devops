pipeline {
    agent { label 'ubuntu-wsl' }

    environment {
        IMAGE_NAME = 'sdineshgandhi/spring-petclinic'
        IMAGE_TAG  = "${BUILD_NUMBER}"

        // Octopus configuration
        OCTOPUS_SERVER_ID = 'octopus-server'
        OCTOPUS_SPACE_ID  = 'Default'
        OCTOPUS_PROJECT   = 'Petclinic'
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
                octopusCreateRelease(
                    serverId: "${OCTOPUS_SERVER_ID}",
                    spaceId: "${OCTOPUS_SPACE_ID}",
                    project: "${OCTOPUS_PROJECT}",
                    releaseVersion: "${BUILD_NUMBER}",
                    toolId: 'Default',
                    deployThisRelease: false,
                    jenkinsUrlLinkback: true,
                    releaseNotes: false,
                    verboseLogging: true
                )
            }
        }
    }

    post {
        success {
            echo "Petclinic pipeline completed successfully."
            echo "Docker image: ${IMAGE_NAME}:${IMAGE_TAG}"
            echo "Octopus release: ${BUILD_NUMBER}"
        }

        failure {
            echo "Pipeline failed. Check the failed stage in the Jenkins console."
        }
    }
}
