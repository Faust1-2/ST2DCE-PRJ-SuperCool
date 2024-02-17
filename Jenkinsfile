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
      sh 'kubectl apply -f k8s/development.yml'
    }
  }
  stage('Check deployment status') {
    // Try to curl localhost:8082
    def status = sh(script: 'curl --silent --output /dev/null --write-out "%{http_code}" localhost:8082', returnStatus: true)

    // Check the status
    if (status == 0) {
        echo 'ok'
        stage('OK Stage') {
            steps {
                echo 'ok'
            }
        }
    } else {
        error('fails')
    }
  }
}