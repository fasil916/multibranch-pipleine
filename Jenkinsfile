pipeline {
  agent any

  stages {
    stage('Build') {
      steps {
        echo "🔨 Building application..."
      }
    }

    stage('Deploy to Dev EKS') {
      when {
        branch 'dev'
      }
      steps {
        echo "🚀 Deploying to Dev EKS cluster"
        // sh 'kubectl apply -f k8s/dev/deployment.yaml'
      }
    }

    stage('Deploy to UAT EKS') {
      when {
        branch 'uat'
      }
      steps {
        echo "🚀 Deploying to UAT EKS cluster"
        // sh 'kubectl apply -f k8s/uat/deployment.yaml'
      }
    }

    stage('Deploy to Prod EKS') {
      when {
        branch 'prod'
      }
      steps {
        echo "🚀 Deploying to Prod EKS cluster"
        // sh 'kubectl apply -f k8s/prod/deployment.yaml'
      }
    }
  }
}

