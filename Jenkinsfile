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
          echo "🛠️ Building Docker image..."
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
          script {
            echo "📦 Pushing image to DockerHub..."
            sh """
              echo "$PASS" | docker login -u "$USER" --password-stdin
              docker push ${IMAGE}:${TAG}
            """
          }
        }
      }
    }

    stage('Deploy to Kubernetes with Helm') {
      agent {
        docker {
          image 'alpine/helm:3.12.0'
          args '-u 0:0' // Agar bisa akses volume dan file kubeconfig
        }
      }
      steps {
        withCredentials([file(credentialsId: "${KUBECONFIG_CRED}", variable: 'KUBE_FILE')]) {
          script {
            echo "🚀 Deploying with Helm..."
            sh """
              echo '📁 Listing kubeconfig file...'
              ls -la "${KUBE_FILE}"

              export KUBECONFIG="${KUBE_FILE}"

              echo '📦 Helm version:'
              helm version

              echo '🚀 Running Helm upgrade/install...'
              helm upgrade --install "${HELM_RELEASE}" ./helm \
                --set image.repository="${IMAGE}" \
                --set image.tag="${TAG}" \
                --namespace "${NAMESPACE}" \
                --create-namespace
            """
          }
        }
      }
    }
  }

  post {
    success {
      echo "✅ Pipeline berhasil! Aplikasi dideploy ke Kubernetes."
    }
    failure {
      echo "❌ Pipeline gagal! Silakan cek log error di atas."
    }
  }
}
