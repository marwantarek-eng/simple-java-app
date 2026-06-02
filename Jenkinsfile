pipeline {
    agent any
    
    environment {
        DOCKER_REGISTRY_USER = 'marwantarek' 
        IMAGE_NAME           = 'simple-java-app'
        IMAGE_TAG            = "${BUILD_NUMBER}"
        DOCKER_HUB_CREDS     = credentials('809a68c7-6c14-4536-82d7-98daff0cd233')
        
        // السطر السحري الجديد
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
            // زودنا -f simple-java-app/pom.xml عشان يشاور على الملف صح
            sh 'docker run --rm -v /var/jenkins_home/.m2:/root/.m2 -v "$(pwd)":/app -w /app maven:3.8.6-openjdk-11 mvn clean package -f simple-java-app/pom.xml -DskipTests'
        }
    }
    
    stage('Test') {
        steps {
            echo 'Running Unit Tests...'
            // زودنا -f simple-java-app/pom.xml هنا كمان
            sh 'docker run --rm -v /var/jenkins_home/.m2:/root/.m2 -v "$(pwd)":/app -w /app maven:3.8.6-openjdk-11 mvn test -f simple-java-app/pom.xml'
        }
    }

        stage('Push to Docker Hub') {
            steps {
                echo 'Building Docker Image and Pushing to Registry...'
                sh '''
                # عمل الـ Docker Image (تأكد إن فيه Dockerfile في الريبو)
                docker build -t $DOCKER_REGISTRY_USER/$IMAGE_NAME:$IMAGE_TAG .
                docker tag $DOCKER_REGISTRY_USER/$IMAGE_NAME:$IMAGE_TAG $DOCKER_REGISTRY_USER/$IMAGE_NAME:latest
                
                # تسجيل الدخول والـ Push أوتوماتيك بالـ Credentials
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
                # لو فيه كونتينر قديم شغال بنفس الاسم بنشيله عشان ميعملش Conflict
                docker stop simple-java-app-running || true
                docker rm simple-java-app-running || true
                
                # تشغيل الـ Image الجديدة اللي لسه عملنالها Push
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
