pipeline {
    agent any  // Use any agent (Docker will be handled in stages)

    environment {
        REACT_APP_NAME = 'shopping-cart'  // Match package.json name
        DOCKER_IMAGE   = 'shopping-cart'  // Docker image name
    }

    stages {
        stage('Checkout Code') {
            steps {
                git(
                    branch: 'main',
                    credentialsId: 'github-token',  // Use correct credentials
                    url: 'https://github.com/prashanty3/reactecommerce.git'
                )
            }
        }

        stage('Install and Build') {
            agent {
                docker {
                    image 'node:18'  // Explicitly use Node 18
                    args '--user root'  // Avoid permission issues
                }
            }
            steps {
                sh 'npm install'
                sh 'npm run build'
            }
        }

        stage('Docker Build & Deploy') {
            steps {
                script {
                    // Ensure Dockerfile exists in repo
                    if (!fileExists('Dockerfile')) {
                        writeFile(
                            file: 'Dockerfile',
                            text: '''
                                FROM nginx:alpine
                                COPY build/ /usr/share/nginx/html
                                EXPOSE 80
                            '''
                        )
                    }
                    sh "docker build -t ${DOCKER_IMAGE} ."
                    sh "docker run -d -p 4000:80 ${DOCKER_IMAGE}"  // Avoid port 3000 conflict
                }
            }
        }
    }
}
