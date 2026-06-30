pipeline {
    agent any

    tools {
        nodejs 'NodeJS_22'
    }

    stages {
        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }
        stage('Run Tests unit and coverage') {
            steps {
                withEnv(['VITE_API_URL=http://localhost:3001']) {
                    sh 'npm run test && npm run test:coverage'
                }
            }
        }
        stage('Publish tests on Jenkins') {
            steps {
                junit '**/reports/junit.xml'
            }
        }
        stage('Analysis with SonarQube') {
            steps {
                withSonarQubeEnv('sonar-qube') {
                    sh 'npm run sonar:scan'
                }
            }
        }
        stage('Build Docker image') {
            steps {
                sh 'docker build -t cicd-tasklist-frontend .'
            }
        }
        stage('Scan image with Trivy') {
            steps {
                sh 'docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy:latest image cicd-tasklist-frontend'
            }
        }
        stage('Generate SBOM with Trivy') {
            steps {
                sh 'mkdir -p reports'
                sh 'docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy:latest -q image --format cyclonedx cicd-tasklist-frontend > reports/sbom.json'
            }
        }
        stage('Push Docker image to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'emmanuelle-docker-password', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    sh 'echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin'
                    sh 'docker tag cicd-tasklist-frontend $DOCKER_USERNAME/cicd-tasklist-frontend:latest'
                    sh 'docker push $DOCKER_USERNAME/cicd-tasklist-frontend:latest'
                }
            }
        }
    }
    post {
        always {
            cleanWs()
        }
    }
}