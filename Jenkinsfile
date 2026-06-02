pipeline {
    agent any

    tools {
        maven 'M3916'
    }
    
    environment {
        DOCKER_REGISTRY_USER = 'marwatarek' // شيلنا الـ n هنا عشان تطابق الأكونت الحقيقي
        IMAGE_NAME           = 'simple-java-app'
        IMAGE_TAG            = "${BUILD_NUMBER}"
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
                sh "docker build -t marwatarek/simple-java-app:\${BUILD_NUMBER} ."
                sh "docker tag marwatarek/simple-java-app:\${BUILD_NUMBER} marwatarek/simple-java-app:latest"
                
                // سطر اللوجين المتظبط باليوزر الصح من غير n والتوكن الشغال مية مية
                sh "docker login -u marwatarek -p dckr_pat_dGKfkPtWxEXNnLxTJ2KGP1UROVw"
                
                sh "docker push marwatarek/simple-java-app:\${BUILD_NUMBER}"
                sh "docker push marwatarek/simple-java-app:latest"
            }
        }

        stage('5. Deploy') {
            steps {
                echo 'Deploying Application to Production...'
                sh '''
                docker stop simple-java-app-running || true
                docker rm simple-java-app-running || true
                docker run -d -p 8081:8080 --name simple-java-app-running marwatarek/simple-java-app:latest
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
