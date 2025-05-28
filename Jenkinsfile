pipeline {
    agent any

    environment {
        REACT_APP_NAME = 'my-react-app'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/prashanty3/reactecommerce.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Build React App') {
            steps {
                sh 'npm run build'
            }
        }

        stage('Docker Build and Run') {
            steps {
                sh '''
                    docker build -t $REACT_APP_NAME .
                    docker run -d -p 3000:80 $REACT_APP_NAME
                '''
            }
        }
    }
}
