pipeline {
  agent any

  environment {
    IMAGE = "sultan877/demo-app"
    TAG = "latest"
    DOCKER_CRED = "docker-hub"
    KUBECONFIG_CRED = "kubeconfig-dev"
    NAMESPACE = "default"
    HELM_RELEASE = "casestudy-jenkins1"
  }

  stages {
    stage('Checkout Source Code') {
      steps {
        git url: 'https://github.com/opick-wq/casestudy-jenkins.git', branch: 'main'
      }
    }

    stage('Build Docker Image') {
      steps {
        script {
          docker.build("${IMAGE}:${TAG}")
        }
      }
    }

    stage('Push Docker Image') {
      steps {
        withCredentials([usernamePassword(
          credentialsId: "${DOCKER_CRED}",
          usernameVariable: 'USER',
          passwordVariable: 'PASS'
        )]) {
          sh '''
            echo "$PASS" | docker login -u "$USER" --password-stdin
            docker push '"${IMAGE}:${TAG}"'
          '''
        }
      }
    }

    stage('Deploy to Kubernetes with Helm') {
      agent {
        docker {
          image 'alpine/helm:3.12.0'
          args '-u 0:0'
        }
      }
      steps {
        withCredentials([file(credentialsId: "${KUBECONFIG_CRED}", variable: 'KUBECONFIG')]) {
          sh '''
            export KUBECONFIG=$KUBECONFIG

            echo "✅ KUBECONFIG location: $KUBECONFIG"
            ls -l $KUBECONFIG

            helm version
            helm upgrade --install casestudy-jenkins1 ./helm \
              --set image.repository=sultan877/demo-app \
              --set image.tag=latest \
              --namespace default \
              --create-namespace
          '''
        }
      }
    }
  }

  post {
    success {
      echo "✅ Deployment sukses!"
    }
    failure {
      echo "❌ Deployment gagal! Cek log error."
    }
  }
}
