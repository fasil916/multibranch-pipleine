pipeline {
  agent any
 environment {
                    // SONAR_TOKEN = credentials('sonar-cred-latest')
                    SONAR_TOKEN = credentials('sonar-new-cred')
                    REGISTRY = '311141542585.dkr.ecr.us-east-1.amazonaws.com'
                    IMAGE_NAME = 'spring-boot-app'
                    // NEWRELIC_API_KEY = credentials('newrelic-api-key')
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
 stage('sonar') {
                steps {
                     echo "Checking vulenerabilties..."
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
      stage('docker  build') {
                steps {
                        echo "🔨 Building ..images....."
                    dir('java-maven-sonar-argocd-helm-k8s/spring-boot-app') {
                         sh 'docker build -t spring-boot-app .'
                         sh "docker tag spring-boot-app ${REGISTRY}/${IMAGE_NAME}:${env.BUILD_NUMBER}"
                    }
                }
            }
    stage('docke rpush') {
                steps {
                  echo "🔨 pusing images..."
                    dir('java-maven-sonar-argocd-helm-k8s/spring-boot-app') {
                         script {
                       docker.withRegistry("https://${REGISTRY}", "ecr:us-east-1:aws-cred") {
                    docker.image("${REGISTRY}/${IMAGE_NAME}:${env.BUILD_NUMBER}").push()
                       }
                         }
                    
                    
                }
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

