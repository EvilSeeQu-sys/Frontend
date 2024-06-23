pipeline {
    agent {
        label 'agent'
    }

    environment {
        PIP_BREAK_SYSTEM_PACKAGES = 1
        scannerHome = tool 'SonarQube'
        dockerRegistry = ''
        registryCredentials = 'dockerhub'
        imageName = "evilseequsys/frontend"
    }

    stages {
        stage('Get Code') {
            steps {
                git branch: 'main', url: 'https://github.com/EvilSeeQu-sys/Frontend'
            }
        }

        stage('Unit tests') {
            steps {
                sh "pip3 install -r requirements.txt"
                sh "python3 -m pytest --cov=. --cov-report xml:test-results/coverage.xml --junitxml=test-results/pytest-report.xml"
            }
        }

        stage('Sonarqube analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh "${scannerHome}/bin/sonar-scanner"
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    dockerTag = "RC-${env.BUILD_NUMBER}"
                    def dockerfileContent = """
                    FROM python:3.9-slim
                    WORKDIR /app
                    COPY requirements.txt .
                    RUN pip install -r requirements.txt
                    COPY . .
                    CMD ["python3", "app.py"]
                    """
                    writeFile file: 'Dockerfile', text: dockerfileContent
                    sh "docker build -t ${imageName}:${dockerTag} ."
                    sh "docker tag ${imageName}:${dockerTag} ${imageName}:latest"
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry("http://${dockerRegistry}", "$registryCredentials") {
                        sh "docker push ${imageName}:${dockerTag}"
                        sh "docker push ${imageName}:latest"
                    }
                }
            }
        }
    }
    
    post {
        always {
            junit 'test-results/*.xml'
            cleanWs()
        }
    }
}
