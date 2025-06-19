pipeline {
  agent any

  environment {
    SONAR_TOKEN = credentials('sonar-new-cred')
    REGISTRY = '311141542585.dkr.ecr.us-east-1.amazonaws.com'
    IMAGE_NAME = 'spring-boot-app'
    IMAGE_TAG  ="${env.BRANCH_NAME}-${env.BUILD_NUMBER}"
  }

  stages {
    stage('Build') {
      steps {
        echo "🔨 Building application..."
        dir('java-maven-sonar-argocd-helm-k8s/spring-boot-app') {
          sh 'mvn clean install'
        }
      }
    }

    stage('SonarQube Scan') {
      steps {
        echo "🔎 Running SonarQube scan..."
        dir('java-maven-sonar-argocd-helm-k8s/spring-boot-app') {
          withSonarQubeEnv('sonar-server') {
            sh '''mvn clean verify sonar:sonar \
              -Dsonar.projectKey=cicd \
              -Dsonar.projectName='cicd' \
              -Dsonar.host.url=http://localhost:9000 \
              -Dsonar.token=${SONAR_TOKEN}'''
          }
        }
      }
    }

    stage('Docker Build') {
      steps {
        echo "🐳 Building Docker image..."
        dir('java-maven-sonar-argocd-helm-k8s/spring-boot-app') {
          
          sh 'docker build -t spring-boot-app .'
          sh "docker tag spring-boot-app ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}"
        }
      }
    }

    stage('Docker Push') {
      steps {
        echo "📤 Pushing Docker image to ECR..."
        echo "${IMAGE_TAG}"
        dir('java-maven-sonar-argocd-helm-k8s/spring-boot-app') {
          script {
            docker.withRegistry("https://${REGISTRY}", "ecr:us-east-1:aws-cred") {
              docker.image("${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}").push()
            }
          }
        }
      }
    }

    stage('Update Manifest File') {
      steps {
        echo "📄 Updating manifest with image tag..."
        dir('java-maven-sonar-argocd-helm-k8s/spring-boot-app-manifests') {
          sh 'cat deployment.yml'
          sh "sed -i 's|replaceImageTag|${IMAGE_TAG}|' deployment.yml"
          sh 'cat deployment.yml'
        }
      }
    }

    stage('Deploy to Dev EKS') {
      when {
        branch 'dev'
      }
      steps {
        echo "🚀 Deploying to Dev EKS cluster"
        dir('java-maven-sonar-argocd-helm-k8s/spring-boot-app-manifests') {
          withKubeCredentials(kubectlCredentials: [[
            caCertificate: '',
            clusterName: 'dev-cluster',
            contextName: 'dev-context',
            credentialsId: 'k8s-cred-dev',
            namespace: 'dev',
            serverUrl: 'https://<your-dev-eks-endpoint>'
          ]]) {
            sh 'kubectl apply --validate=false -f deployment.yml'
            sh "kubectl rollout status deployment/spring-boot-app --timeout=120s"
          }
        }
      }
    }

    stage('Deploy to UAT EKS') {
      when {
        branch 'uat'
      }
      steps {
        echo "🚀 Deploying to UAT EKS cluster"
        dir('java-maven-sonar-argocd-helm-k8s/spring-boot-app-manifests') {
          withKubeCredentials(kubectlCredentials: [[
            caCertificate: '',
            clusterName: 'minikube',
            contextName: 'minikube',
            credentialsId: 'k8s-cred',
            namespace: 'uat',
            serverUrl: 'https://127.0.0.1:32771'
          ]]) {
            sh 'kubectl apply --validate=false -f deployment.yml'
           
          }
        }
      }
    }

    stage('Deploy to Prod EKS') {
      when {
        branch 'prod'
      }
      steps {
        echo "🚀 Deploying to Prod EKS cluster"
        dir('java-maven-sonar-argocd-helm-k8s/spring-boot-app-manifests') {
          withKubeCredentials(kubectlCredentials: [[
            caCertificate: '',
            clusterName: 'minikube',
            contextName: 'minikube',
            credentialsId: 'k8s-cred',
            namespace: 'default',
            serverUrl: 'https://127.0.0.1:32771'
          ]]) {
            sh 'kubectl apply --validate=false -f deployment.yml'
           
          }
        }
      }
    }
  }
}
