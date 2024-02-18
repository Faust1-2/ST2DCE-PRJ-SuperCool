node {
  stage('checkout') {
    checkout scm
  }

  try {
    stage('build docker image') {
      sh 'docker build -t supercool-docker:${BUILD_ID} --build-arg VARIABLE=${BUILD_ID} .'
    }
  
    stage('Deploy to development') {
      withKubeConfig([
        credentialsId: 'kubernetes-credentials',
        serverUrl: 'https://localhost:6443'    
      ]) {
        sh 'helm install deployment-${BUILD_ID} deployment/ --set image.tag=${BUILD_ID} --set metadata.namespace=development'
      }
    }

    stage('Check deployment status') {
      sleep(5) // Wait for the deployment to be ready
      // Try to curl localhost:8082
      def status = sh(script: 'curl --silent --output /dev/null --write-out "%{http_code}" localhost:8082', returnStatus: true)

      // Check the status
      if (status == 200 ) {
        stage('Deploy to production') {
          withKubeConfig([
            credentialsId: 'kubernetes-credentials',
            serverUrl: 'https://localhost:6443'    
          ]) {
            sh 'helm upgrade deployment-${BUILD_ID} deployment/ --set image.tag=${BUILD_ID} --set metadata.namespace=production'
          }
        }
      } else {
        error('fails')
      }
    }
  } catch (Exception e) {
    currentBuild.result = 'FAILURE'
    echo 'Deployment failed'
    stage('cleanup Development') {
      withKubeConfig([
        credentialsId: 'kubernetes-credentials',
        serverUrl: 'https://localhost:6443'    
      ]) {
        echo 'Deleting deployment...'
        sh 'helm uninstall deployment-${BUILD_ID}'
        echo 'Deleted'
      }
    }
    throw e
  }
}