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

        stage('Install Docker Without get.docker.com Using apt') {
            steps {
                script {
                    def dockerInstalled = sh(script: 'which docker', returnStatus: true) == 0
                    if (!dockerInstalled) {
                        echo '🔧 Docker not found. Installing via official APT repo using apt...'
                        sh '''
                            sudo apt update -y
                            sudo apt install -y ca-certificates curl gnupg lsb-release
        
                            sudo install -m 0755 -d /etc/apt/keyrings
                            sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
                            sudo chmod a+r /etc/apt/keyrings/docker.asc
        
                            echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] \
                            https://download.docker.com/linux/ubuntu \
                            $(. /etc/os-release && echo \\"${UBUNTU_CODENAME:-$VERSION_CODENAME}\\") stable" | \
                            sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
        
                            sudo apt update -y
                            sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
        
                            sudo systemctl enable docker
                            sudo systemctl start docker
                        '''
                    } else {
                        echo '✅ Docker is already installed.'
                        sh 'docker --version'
                    }
                }
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
                // 1. First update browserslist data
                sh 'npx update-browserslist-db@latest'
    
                // 2. Install all dependencies including the required Babel plugin
                sh 'npm install --save-dev @babel/plugin-proposal-private-property-in-object'
                sh 'npm install'
    
                // 3. Run the build
                sh 'CI=false npm run build'  // Temporarily disable CI checks if needed
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
