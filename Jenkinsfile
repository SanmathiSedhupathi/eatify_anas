pipeline {
    agent any

    environment {
        DOCKERHUB_USERNAME = 'sanmathisedhupathi'
        DOCKERHUB_PASSWORD = '08-Sep-2004'
        IMAGE_NAME = 'sanmathisedhupathi/eatify'
        EMAIL_RECIPIENTS = 'sanmathisedhupathi2004@gmail.com' // change to your email
    }

    stages {
        

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $IMAGE_NAME:${BUILD_NUMBER} .'
            }
        }

        stage('Push to Docker Hub') {
            steps {
                sh "echo $DOCKERHUB_PASSWORD | docker login -u $DOCKERHUB_USERNAME --password-stdin"
                sh "docker push $IMAGE_NAME:${BUILD_NUMBER}"
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    writeFile file: 'k8s-deployment.yaml', text: """
apiVersion: apps/v1
kind: Deployment
metadata:
  name: eatify-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: eatify
  template:
    metadata:
      labels:
        app: eatify
    spec:
      containers:
      - name: eatify
        image: $IMAGE_NAME:${BUILD_NUMBER}
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: eatify-service
spec:
  type: LoadBalancer
  selector:
    app: eatify
  ports:
    - port: 80
      targetPort: 80
"""
                    sh 'kubectl apply -f k8s-deployment.yaml'
                }
            }
        }
    }

    post {
        success {
            emailext(
                subject: "✅ SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: "Build succeeded!\n\nCheck it here: ${env.BUILD_URL}",
                to: "${env.EMAIL_RECIPIENTS}"
            )
        }
        failure {
            emailext(
                subject: "❌ FAILURE: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: "Build failed.\n\nCheck it here: ${env.BUILD_URL}",
                to: "${env.EMAIL_RECIPIENTS}"
            )
        }
    }
}
