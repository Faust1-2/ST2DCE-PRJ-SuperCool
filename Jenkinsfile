node {
  stage('build docker image') {
    sh 'docker build -t supercool-:${BUILD_ID} .'
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