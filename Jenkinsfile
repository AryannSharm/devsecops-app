pipeline {
  agent any
  environment {
    IMAGE  = "healthcare-app"
    TAG    = "${env.BUILD_NUMBER}"
    TMPDIR = "/var/lib/jenkins/trivy-tmp"
  }
  stages {
    stage('Checkout') {
      steps { checkout scm }
    }
    stage('Build image') {
      steps {
        sh '''
          sed -i "s/__BUILD__/$TAG/g" index.html
          docker build -t $IMAGE:$TAG .
        '''
      }
    }
    stage('Security scan (Trivy gate)') {
      steps { sh 'trivy image --scanners vuln --exit-code 1 --severity HIGH,CRITICAL $IMAGE:$TAG' }
    }
    stage('Deploy') {
      steps {
        sh '''
          kind load docker-image $IMAGE:$TAG --name devops
          sed "s/IMAGE_TAG/$TAG/" k8s/deployment.yml | kubectl apply -f -
          kubectl rollout status deployment/healthcare-app --timeout=120s
        '''
      }
    }
  }
}
