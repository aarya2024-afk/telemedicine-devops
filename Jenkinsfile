pipeline {
    agent any

    options {
        skipDefaultCheckout(true)
        timestamps()
    }

    environment {
        DOCKERHUB_USER = 'aarya120924'
        IMAGE_NAME = 'telemedicine'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Validate Application') {
            steps {
                sh '''
                    test -f index.html
                    test -f style.css
                    test -f Dockerfile
                    test -f Jenkinsfile
                    echo "Application files validated successfully."
                '''
            }
        }

        stage('Set Version') {
            when {
                expression {
                    return env.BRANCH_NAME in ['v1', 'v2', 'v3']
                }
            }
            steps {
                script {
                    env.VERSION = env.BRANCH_NAME
                    env.IMAGE = "${DOCKERHUB_USER}/${IMAGE_NAME}:${VERSION}"
                }

                echo "Building version: ${env.VERSION}"
                echo "Docker image: ${env.IMAGE}"
            }
        }

        stage('Build Docker Image') {
            when {
                expression {
                    return env.BRANCH_NAME in ['v1', 'v2', 'v3']
                }
            }
            steps {
                sh '''
                    docker build --pull -t "$IMAGE" .
                '''
            }
        }

        stage('Push to Docker Hub') {
            when {
                expression {
                    return env.BRANCH_NAME in ['v1', 'v2', 'v3']
                }
            }
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds-aarya',
                        usernameVariable: 'DH_USER',
                        passwordVariable: 'DH_TOKEN'
                    )
                ]) {
                    sh '''
                        set +x
                        echo "$DH_TOKEN" | docker login \
                            --username "$DH_USER" \
                            --password-stdin

                        docker push "$IMAGE"
                        docker logout
                    '''
                }
            }
        }

        
stage('Deploy to Kubernetes') {
    when {
        expression {
            return env.BRANCH_NAME in ['v1', 'v2', 'v3']
        }
    }
    steps {
        sh '''
            kubectl set image \
                "deployment/telemedicine-${VERSION}" \
                "web=${IMAGE}"

            kubectl rollout status \
                "deployment/telemedicine-${VERSION}" \
                --timeout=180s
        '''
    }
}

    }

    post {
        success {
            echo "Pipeline completed successfully for branch ${env.BRANCH_NAME}."
        }

        failure {
            echo "Pipeline failed for branch ${env.BRANCH_NAME}. Check the stage logs."
        }
    }
}