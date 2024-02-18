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
        sh 'helm install development-${BUILD_ID} deployment/ --set image.tag=${BUILD_ID} --set metadata.namespace=development'
      }
    }

    stage('Check deployment status') {
      sleep(10) // Wait for the deployment to be ready
      // Try to curl localhost:8082
      final String status = sh(script: 'curl --silent --output /dev/null --write-out "%{http_code}" localhost:80', returnStdout: true)
      
      echo status

      // Check the status
      if (status == "200") {
        stage('Deploy to production') {
          withKubeConfig([
            credentialsId: 'kubernetes-credentials',
            serverUrl: 'https://localhost:6443'    
          ]) {
            sh 'helm upgrade --install production deployment/ --set image.tag=${BUILD_ID} --set metadata.namespace=production'
          }
        }
      } else {
        error('Could not get a response from development')
      }
    }
  } catch (Exception e) {
    currentBuild.result = 'FAILURE'
    echo 'Deployment failed'
    throw e
  } finally {
      stage('cleanup Development') {
      withKubeConfig([
        credentialsId: 'kubernetes-credentials',
        serverUrl: 'https://localhost:6443'    
      ]) {
        echo 'Deleting deployment...'
        sh 'helm uninstall development-${BUILD_ID}'
        echo 'Deleted'
      }
    }
  }
}