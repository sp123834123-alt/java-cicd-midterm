pipeline {
    agent any

    environment {
        SONAR_HOST_URL = 'http://localhost:9000'
        SONAR_TOKEN = credentials('sonar-token')
        DOCKER_IMAGE = 'SECRETAGENT_2Z/java-app:latest'
        DOCKER_CREDENTIALS_ID = 'dockerhub-creds'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/spring-petclinic/spring-framework-petclinic.git'
            }
        }

        stage('Build with Java 17') {
            steps {
                script {
                    docker.image('maven:3.9.6-eclipse-temurin-17').inside('--network host') {
                        sh 'MAVEN_OPTS="-Xmx512m" mvn -Dmaven.repo.local=/tmp/m2repo clean package -DskipTests'
                    }
                }
            }
        }

        stage('Test with Java 17') {
            steps {
                script {
                    docker.image('maven:3.9.6-eclipse-temurin-17').inside('--network host') {
                        sh 'MAVEN_OPTS="-Xmx512m" mvn -Dmaven.repo.local=/tmp/m2repo test'
                    }
                }
            }
        }

        stage('Static Code Analysis with Java 17') {
            steps {
                script {
                    docker.image('maven:3.9.6-eclipse-temurin-17').inside('--network host') {
                        sh """
                        MAVEN_OPTS="-Xmx512m" mvn -Dmaven.repo.local=/tmp/m2repo sonar:sonar \
                          -Dsonar.projectKey=java-app \
                          -Dsonar.host.url=${SONAR_HOST_URL} \
                          -Dsonar.login=${SONAR_TOKEN}
                        """
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t ${DOCKER_IMAGE} .'
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', DOCKER_CREDENTIALS_ID) {
                        sh 'docker push ${DOCKER_IMAGE}'
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh "sed -i 's|secretagent_2z/java-app:latest|${DOCKER_IMAGE}|g' deployment.yaml"
                sh 'kubectl apply -f deployment.yaml'
            }
        }
    }
}
