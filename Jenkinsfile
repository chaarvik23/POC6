pipeline {
    agent any

    environment {
        SONARQUBE = 'SonarQubeServer'
        SONAR_PROJECT_KEY = 'POC6'
        DOCKER_IMAGE = 'myapp:latest'
        SONAR_HOST_URL = 'http://20.51.208.16/:9000'
    }

    stages {
        stage('Checkout') {
            steps {
                // Clone the repository from GitHub
                git branch: 'main', url: 'https://github.com/chaarvik23/POC6.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean install -DskipTests -Ddependency-check.skip=true'
            }
        }

        stage('SonarQube Analysis') {
            environment {
                SONAR_TOKEN = credentials('sonar') // Must exist in Jenkins credentials
            }
            steps {
                withSonarQubeEnv('My SonarQube Server') {
                    sh """
                        mvn sonar:sonar \
                          -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                          -Dsonar.host.url=${SONAR_HOST_URL} \
                          -Dsonar.login=${SONAR_TOKEN}
                    """
                }
            }
        }


        stage('Docker Build') {
            steps {
                sh 'docker build -t ${DOCKER_IMAGE} .'
            }
        }

        stage('Image Scan with Trivy') {
            steps {
                sh 'trivy image ${DOCKER_IMAGE} || true'
            }
        }

        stage('Run Container') {
            steps {
                sh 'docker run -d -p 8085:8080 ${DOCKER_IMAGE}'
            }
        }
    }
}
