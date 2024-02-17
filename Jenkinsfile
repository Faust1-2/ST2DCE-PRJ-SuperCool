node {

  environment {
    DOCKER_ID = "${BUILD_ID}"
  }
  try {
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
      if (status == 200 ) {
        success('works')
      } else {
        error('fails')
      }
    }
  } catch (Exception e) {
    currentBuild.result = 'FAILURE'
    echo 'Development deployment failed'
    throw e
  } finally {
    stage('cleanup') {
      echo 'Deleting deployment...'
      sh 'kubectl delete -f k8s/development.yml'
      echo 'Deleted'
      echo 'Deleting docker image...'
      sh 'docker rmi supercool-docker:${BUILD_ID}'
      echo 'Deleted'
    }
  }
}