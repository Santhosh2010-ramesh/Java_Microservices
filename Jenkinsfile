pipeline {
    agent any

    environment {
        REGISTRY = "santhosh2010/java_micro"
        IMAGE_TAG = "${env.BRANCH_NAME}-${env.BUILD_NUMBER}"
        SONARQUBE_ENV = "SonarQube"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build & Test') {
            steps {
                sh 'mvn clean install'
            }
        }

        stage('SonarQube Analysis') {
            when {
                anyOf {
                    branch pattern: "feature/.*", comparator: "REGEXP"
                    branch 'develop'
                    branch 'main'
                }
            }
            steps {
                withSonarQubeEnv("${SONARQUBE_ENV}") {
                    sh 'mvn sonar:sonar'
                }
            }
        }

        stage('Quality Gate') {
            when {
                anyOf {
                    branch pattern: "feature/.*", comparator: "REGEXP"
                    branch 'develop'
                    branch 'main'
                }
            }
            steps {
                timeout(time: 1, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Docker Build & Push') {
            when {
                anyOf {
                    branch 'develop'
                    branch 'main'
                }
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    sh """
                        docker build -t $REGISTRY:$IMAGE_TAG .
                        echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin
                        docker push $REGISTRY:$IMAGE_TAG
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            when {
                anyOf {
                    branch 'develop'
                    branch 'main'
                }
            }
            steps {
                sh """
                    kubectl apply -f k8s/deployment.yaml
                    kubectl apply -f k8s/service.yaml
                """
            }
        }
    }
}
