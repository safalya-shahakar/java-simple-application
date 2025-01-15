pipeline {
    agent {
        docker { image 'maven:3.8.5-jdk-11' }
    }
    environment {
        DOCKER_CREDENTIALS = credentials('dockerhub-credentials-id')  // Set this in Jenkins
    }
    stages {
        stage('Checkout Code') {
            steps {
                git 'https://github.com/yourusername/yourrepo.git' // Replace with your repo URL
            }
        }
        stage('Build with Maven') {
            steps {
                sh 'mvn clean install'
            }
        }
        stage('Code Analysis with SonarQube') {
            steps {
                withSonarQubeEnv('SonarQube') { // Configure SonarQube in Jenkins
                    sh 'mvn sonar:sonar'
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t yourdockerhubusername/yourimage:latest .'
            }
        }
        stage('Push to DockerHub') {
            steps {
                sh '''
                echo $DOCKER_CREDENTIALS | docker login -u yourdockerhubusername --password-stdin
                docker push yourdockerhubusername/yourimage:latest
                '''
            }
        }
    }
    post {
        always {
            echo 'Pipeline finished'
        }
    }
}
