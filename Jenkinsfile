pipeline {
    agent any

    tools {
        // بنعرف جينكنز يستخدم أداة المافن اللي متسطبة في الـ Global Tools
        maven 'M3916'
    }
    
    environment {
        DOCKER_REGISTRY_USER = 'marwantarek' 
        IMAGE_NAME           = 'simple-java-app'
        IMAGE_TAG            = "${BUILD_NUMBER}"
        // الـ ID بتاع الـ Credentials الرخم بتاعك
        DOCKER_HUB_CREDS     = credentials('809a68c7-6c14-4536-82d7-98daff0cd233')
        // الـ TCP endpoint عشان الدوكر اللي جوه جينكنز يكلم الـ VM بره
        DOCKER_HOST          = 'tcp://172.17.0.1:2375'
    }

    stage('1. Fetch Code') {
            steps {
                echo 'Fetching Code from Your GitHub Repo...'
                // غيرنا الرابط هنا للريبو بتاعك أنت يا برنس
                git branch: 'main', url: 'https://github.com/marwantarek-eng/simple-java-app.git'
            }
        }

        stage('2. Build') {
            steps {
                echo 'Building Java Application using Maven...'
                // جينكنز هيشغل المافن علطول جوه الـ Workspace والنصيب شغال لوحده!
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
                sh '''
                # بناء الـ Image بالـ tags بتاعتك يا مروان
                docker build -t $DOCKER_REGISTRY_USER/$IMAGE_NAME:$IMAGE_TAG .
                docker tag $DOCKER_REGISTRY_USER/$IMAGE_NAME:$IMAGE_TAG $DOCKER_REGISTRY_USER/$IMAGE_NAME:latest
                
                # الـ Login والـ Push أوتوماتيك لـ حسابك
                echo $DOCKER_HUB_CREDS_PSW | docker login -u $DOCKER_REGISTRY_USER --password-stdin
                docker push $DOCKER_REGISTRY_USER/$IMAGE_NAME:$IMAGE_TAG
                docker push $DOCKER_REGISTRY_USER/$IMAGE_NAME:latest
                '''
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
