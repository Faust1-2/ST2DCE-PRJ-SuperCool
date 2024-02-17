pipeline {
  agent any
  stages {
    stage('build docker image') {
      steps {
        sh 'docker build -t supercool-:${BUILD_ID} .'
      }
    }
  }
  stage('List pods') {
    withKubeConfig([
        credentialsId: 'kubernetes-credentials',
        serverUrl: 'https://localhost:6443'    
    ]) {
      steps {
        sh 'kubectl apply -f k8s/development.yaml'
      }
    }
  }
}