pipeline {
  agent { label 'bastion' }

  environment {
    REPO_URL       = 'https://github.com/sagardpatil0055/example-voting-app.git'
    AWS_REGION     = 'us-east-1'
    AWS_ACCOUNT_ID = '980104576357'
    ECR_VOTE_REPO  = 'vote'
    ECR_RESULT_REPO = 'result'
    SSH_KEY_PATH   = "${HOME}/project.pem"
    APP_TAG_NAME   = 'bastion'
  }

  parameters {
    string(name: 'BRANCH', defaultValue: 'main', description: 'Git branch to build')
  }

  stages {
    stage('Clone Repository') {
      steps {
        git branch: "${params.BRANCH}", url: "${REPO_URL}"
      }
    }

    stage('Build & Push Vote Image') {
      steps {
        script {
          sh """
            docker build -t vote:latest ./vote
            docker tag vote:latest ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_VOTE_REPO}:latest
            docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_VOTE_REPO}:latest
          """
        }
      }
    }

    stage('Build & Push Result Image') {
      steps {
        script {
          sh """
            docker build -t result:latest ./result
            docker tag result:latest ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_RESULT_REPO}:latest
            docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_RESULT_REPO}:latest
          """
        }
      }
    }

    stage('Fetch App Private IPs') {
      steps {
        script {
          def ipListRaw = sh(
            script: """aws ec2 describe-instances \
              --filters "Name=tag:Name,Values=${APP_TAG_NAME}" "Name=instance-state-name,Values=running" \
              --query "Reservations[*].Instances[*].PrivateIpAddress" \
              --region ${AWS_REGION} --output text""",
            returnStdout: true
          ).trim()

          def ipList = ipListRaw.split("\\s+")
          env.APP_IPS = ipList.join(',')
          echo "📡 App Host IPs: ${env.APP_IPS}"
        }
      }
    }

    stage('Deploy Using Docker Compose') {
      steps {
        script {
          def ipList = env.APP_IPS.split(',')
          for (ip in ipList) {
            echo "🚀 Deploying to ${ip}"
            sh """
              scp -o StrictHostKeyChecking=no -i ${SSH_KEY_PATH} docker-compose.yml ubuntu@${ip}:/home/ubuntu/
            
              ssh -o StrictHostKeyChecking=no -i ${SSH_KEY_PATH} ubuntu@${ip} \\
                "docker pull ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_VOTE_REPO}:latest && \\
                 docker pull ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_RESULT_REPO}:latest && \\
                 docker compose -f docker-compose.yml down || true && \\
                 docker compose -f docker-compose.yml up -d"
            """
          }
        }
      }
    }
  }

  post {
    success {
      echo "✅ Vote App Stack deployed using Docker Compose"
    }
    failure {
      echo "❌ Deployment failed. Please verify SSH access or ECR login."
    }
  }
}
