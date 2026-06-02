pipeline {
    agent any

    tools {
        maven 'M3916'
    }
    
    environment {
        DOCKER_REGISTRY_USER = 'marwantarek' 
        IMAGE_NAME           = 'simple-java-app'
        IMAGE_TAG            = "${BUILD_NUMBER}"
        DOCKER_HUB_CREDS     = credentials('809a68c7-6c14-4536-82d7-98daff0cd233')
        DOCKER_HOST          = 'tcp://172.17.0.1:2375'
    }

    stages {
        stage('1. Fetch Code') {
            steps {
                echo 'Fetching Code from Your GitHub Repo...'
                git branch: 'main', url: 'https://github.com/marwantarek-eng/simple-java-app.git'
            }
        }

        stage('2. Build') {
            steps {
                echo 'Building Java Application using Maven...'
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('3. Test') {
            steps {
                echo 'Running Unit Tests...'
                sh 'mvn test'
            }
        }

        stage('4. Push (Docker Image)') {
                    steps {
                        echo 'Building Docker Image and Pushing to Docker Hub...'
                        // هنستخدم الـ token بتاعك مانيوال هنا عشان نضمن الـ Auth 100%
                        withCredentials([usernamePassword(credentialsId: 'docker--cred', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                            sh '''
                            docker build -t marwantarek/simple-java-app:${BUILD_NUMBER} .
                            docker tag marwantarek/simple-java-app:${BUILD_NUMBER} marwantarek/simple-java-app:latest
                            
                            # لو الـ credentialsId ملقطش الـ Token الجديد، حط الـ Token الحقيقي مكان الـ Variable ده مانيوال للتجربة
                            echo "$DOCKER_PASS" | docker login -u marwantarek --password-stdin
                            
                            docker push marwantarek/simple-java-app:${BUILD_NUMBER}
                            docker push marwantarek/simple-java-app:latest
                            '''
                        }
                    }
                }

        stage('5. Deploy') {
            steps {
                echo 'Deploying Application to Production...'
                sh '''
                docker stop simple-java-app-running || true
                docker rm simple-java-app-running || true
                docker run -d -p 8081:8080 --name simple-java-app-running $DOCKER_REGISTRY_USER/$IMAGE_NAME:$IMAGE_TAG
                '''
                echo 'Application is live on port 8081!'
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
