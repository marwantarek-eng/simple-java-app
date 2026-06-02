pipeline {
    agent any
    
    environment {
        DOCKER_REGISTRY_USER = 'marwantarek' 
        IMAGE_NAME           = 'simple-java-app'
        IMAGE_TAG            = "${BUILD_NUMBER}"
        DOCKER_HUB_CREDS     = credentials('809a68c7-6c14-4536-82d7-98daff0cd233')
        DOCKER_HOST          = 'tcp://172.17.0.1:2375'
    }

    stages {
        stage('Clone Code') {
            steps {
                echo 'Fetching code from GitHub...'
                git branch: 'main', url: 'https://github.com/HaythamMohamd/simple-java-app.git'
            }
        }

        stage('Build & Package') {
            steps {
                echo 'Building Java Application using Maven Container...'
                // بنقش الـ volumes من حاوية جينكنز علطول عشان نقرأ الـ pom.xml صح
                sh 'docker run --rm --volumes-from jenkins -w /var/jenkins_home/workspace/jenkins-java-iti maven:3.8.6-openjdk-11 mvn clean package -DskipTests'
            }
        }
        
        stage('Test') {
            steps {
                echo 'Running Unit Tests...'
                sh 'docker run --rm --volumes-from jenkins -w /var/jenkins_home/workspace/jenkins-java-iti maven:3.8.6-openjdk-11 mvn test'
            }
        }

        stage('Push to Docker Hub') {
            steps {
                echo 'Building Docker Image and Pushing to Registry...'
                sh '''
                docker build -t $DOCKER_REGISTRY_USER/$IMAGE_NAME:$IMAGE_TAG .
                docker tag $DOCKER_REGISTRY_USER/$IMAGE_NAME:$IMAGE_TAG $DOCKER_REGISTRY_USER/$IMAGE_NAME:latest
                
                echo $DOCKER_HUB_CREDS_PSW | docker login -u $DOCKER_REGISTRY_USER --password-stdin
                docker push $DOCKER_REGISTRY_USER/$IMAGE_NAME:$IMAGE_TAG
                docker push $DOCKER_REGISTRY_USER/$IMAGE_NAME:latest
                '''
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying Application Container locally...'
                sh '''
                docker stop simple-java-app-running || true
                docker rm simple-java-app-running || true
                docker run -d -p 8081:8080 --name simple-java-app-running $DOCKER_REGISTRY_USER/$IMAGE_NAME:$IMAGE_TAG
                '''
                echo 'Application deployed successfully on port 8081!'
            }
        }
    }
    
    post {
        always {
            echo 'Cleaning up docker login session...'
            sh 'docker logout || true'
        }
    }
}
