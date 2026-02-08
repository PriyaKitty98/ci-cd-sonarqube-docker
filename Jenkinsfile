pipeline {
    agent any
    

    environment {
        SONARQUBE_ENV = 'SonarQube'
        DOCKER_IMAGE = 'priyakitty98/ci-cd-sonarqube-docker'
        DOCKERHUB_CREDENTIALS = 'dockerhub-token'
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/PriyaKitty98/ci-cd-sonarqube-docker.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                bat 'pip install -r requirements.txt'
            }
        }

        stage('Test') {
            steps {
                bat 'pytest'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    bat """
                    sonar-scanner ^
                    -Dsonar.projectKey=ci-cd-sonadocker ^
                    -Dsonar.sources=.
            """
                }
            }
        }   

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Docker Build') {
            steps {
                bat 'docker build -t %DOCKER_IMAGE%:latest .'
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: DOCKERHUB_CREDENTIALS,
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    bat """
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    docker push %DOCKER_IMAGE%:latest
                    """
                }
            }
        }

        stage('Deploy') {
            steps {
                bat 'docker rm -f ci-cd-sonarqube-docker || exit 0'
                bat 'docker run -d -p 8081:8080 --name ci-cd-sonarqube-docker %DOCKER_IMAGE%:latest'
            }
        }
    }
}
