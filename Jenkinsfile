pipeline {
    agent any

    environment {
        IMAGE_NAME = 'jin604/biddinggo-be'
        IMAGE_TAG = "${env.BUILD_NUMBER}"
    }

    stages {
        stage('Build Image') {
            steps {
                sh '''
                  docker build \
                    -t $IMAGE_NAME:$IMAGE_TAG \
                    -t $IMAGE_NAME:latest \
                    .
                '''
            }
        }

        stage('Push Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-access', usernameVariable: 'DOCKERHUB_USER', passwordVariable: 'DOCKERHUB_TOKEN')]) {
                    sh '''
                      echo "$DOCKERHUB_TOKEN" | docker login -u "$DOCKERHUB_USER" --password-stdin
                      docker push $IMAGE_NAME:$IMAGE_TAG
                      docker push $IMAGE_NAME:latest
                    '''
                }
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                  cd /volume1/docker/biddinggo/deploy
                  git pull
                  docker-compose pull backend
                  docker-compose up -d backend nginx
                  sleep 15
                  curl -f http://127.0.0.1:5173/actuator/health
                '''
            }
        }
    }

    post {
        success {
            withCredentials([string(credentialsId: 'discord-webhook', variable: 'DISCORD_WEBHOOK_URL')]) {
                sh '''
                  curl -X POST \
                    -H 'Content-Type: application/json' \
                    --data "{
                      \\"content\\": \\"✅ **BiddingGo BE CI/CD Success**\\\\n\\\\n**Job**: ${JOB_NAME}\\\\n**Build**: #${BUILD_NUMBER}\\\\n**Image**: ${IMAGE_NAME}:${IMAGE_TAG}\\\\n**Also Tagged**: ${IMAGE_NAME}:latest\\\\n**Deploy**: NAS Docker Compose\\\\n**Health**: /actuator/health\\\\n**URL**: ${BUILD_URL}\\"
                    }" \
                    "$DISCORD_WEBHOOK_URL"
                '''
            }
        }

        failure {
            withCredentials([string(credentialsId: 'discord-webhook', variable: 'DISCORD_WEBHOOK_URL')]) {
                sh '''
                  curl -X POST \
                    -H 'Content-Type: application/json' \
                    --data "{
                      \\"content\\": \\"❌ **BiddingGo BE CI/CD Failed**\\\\n\\\\n**Job**: ${JOB_NAME}\\\\n**Build**: #${BUILD_NUMBER}\\\\n**Image**: ${IMAGE_NAME}:${IMAGE_TAG}\\\\n**Stage**: Check Jenkins Console Log\\\\n**URL**: ${BUILD_URL}\\"
                    }" \
                    "$DISCORD_WEBHOOK_URL"
                '''
            }
        }
    }
}

