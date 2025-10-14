pipeline {
  agent any

  environment {
    AWS_REGION = 'us-east-1'
    ECR_REPO = '684160548236.dkr.ecr.us-east-1.amazonaws.com/todo-ecr'
    IMAGE_TAG = "${env.BUILD_ID}"
    S3_BUCKET = 'todo-cicd'
   // KUBE_CONFIG = credentials('eks-kubeconfig') // Jenkins credential ID
  }

  stages {
    stage('Checkout SCM') {
      steps {
        git branch: 'main', url: 'https://github.com/your-org/todo-springboot.git'
      }
    }

    stage('Build Application') {
      steps {
        sh './mvnw clean package -DskipTests'
      }
    }

    stage('Build Docker Image') {
      steps {
        script {
          sh """
            aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin ${ECR_REPO}
            docker build -t ${ECR_REPO}:${IMAGE_TAG} .
            docker tag ${ECR_REPO}:${IMAGE_TAG} ${ECR_REPO}:latest
          """
        }
      }
    }

    stage('Deploy to EKS') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-crede-id']]) {
                    sh '''
                    aws eks --region $AWS_REGION update-kubeconfig --name $EKS_CLUSTER
                    kubectl apply -f k8s/deployment.yml
                    kubectl apply -f k8s/service.yml

                    '''
                }
            }
        }
 

    stage('Upload Artifact to S3') {
      steps {
        sh """
          aws s3 cp target/todo-spring.jar s3://${S3_BUCKET}/artifacts/todo-spring-${IMAGE_TAG}.jar
        """
      }
    }

    stage('Deploy to EKS') {
      steps {
        withCredentials([file(credentialsId: 'eks-kubeconfig', variable: 'KUBECONFIG')]) {
          sh """
            aws eks update-kubeconfig --name todo-eks --region us-east-1
            kubectl apply -f k8s/deployment.yaml
            kubectl apply -f k8s/service.yaml
          """
        }
      }
    }
  }

  post {
    success {
      echo '✅ Deployment completed successfully!'
    }
    failure {
      echo '❌ Deployment failed. Check logs for details.'
    }
  }
}
