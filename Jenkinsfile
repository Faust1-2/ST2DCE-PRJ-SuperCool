node {
  stage('checkout') {
    checkout scm
  }
  stage('build docker image') {
    sh 'docker build -t supercool-docker:${BUILD_ID} --build-arg VARIABLE=${BUILD_ID} .'
  }
  stage('Deploy to development') {
    withKubeConfig([
        credentialsId: 'kubernetes-credentials',
        serverUrl: 'https://localhost:6443'    
    ]) {
      sh 'kubectl apply -f k8s/development.yaml'
    }
  }
}