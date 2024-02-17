## Kubernetes secret:

Create a service and its token with: 

```bash
kubectl apply -f serviceAccount.yml
```

Retrieve the token with:

```bash
kubectl get secret jenkins-secret -o jsonpath="{.data.token}" | base64 --decode
```

## Jenkins configuration

### Required plugins

Go to `Manage Jenkins` > `Plugins` > `Available plugins` and search for `Kubernetes CLI`.

### Kubernetes credentials

Go to `Manage Jenkins` > `Credentials` > `globals` and add a new credential:
- Select `Secret text` as the **type** of the secret
- In **secret**, paste the decoded json token of the user.
- In **ID**, paste `jenkins-credentials`.
- Click on **create**

### Create your pipeline

From now on, you can access kubernetes api with:

```jenkins
node {
  stage('List pods') {
    withKubeConfig([
        credentialsId: 'kubernetes-credentials',
        serverUrl: 'https://localhost:6443'    
    ]) {
      // commands...
    }
  }
}
```
