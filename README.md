

# gcp-cloudbuild-substitution-variables

This script enables you to inject secrets into your app engine deployments via Cloud Build Triggers. It assumes you have your secrets defined as a substitution variable(s) inside your Cloud Build trigger and that you add this script as a build step which will modify your `app.yaml` and inject the Substitution Variables you defined as environment variables into your `app.yaml`. If you manage your Cloud Build Triggers using terraform, like we do, then you can keep everything encrypted in source control with a little use of the KMS resources from GCP.




Credits: https://nlbao.page/posts/2021/how-to-set-environment-variables-in-google-app-engine-using-cloud-build/

Below assume you have a Cloud Build Triger with two subsitution variables defined: `_API_CLIENT_ID` and `_HUBSPOT_SYNC_ENABLED` and want them injected into your application as: `API_CLIENT_ID` and `HUBSPOT_SYNC_ENABLED`.

### Example app.yaml

```
runtime: nodejs16
service: default
main: node index.js 
env_variables:
```

***Note: the app.yaml should have env_variables: defined at the very end without a newline (\n)***

### Example cloudbuild.yaml

```
steps:
  - name: launcher.gcr.io/google/ubuntu2004
    env:
      - 'API_CLIENT_ID=${_API_CLIENT_ID}'
      - 'HUBSPOT_SYNC_ENABLED=${_HUBSPOT_SYNC_ENABLED}'
    args:
      - '-c'
      - >-
        curl -s
        https://raw.githubusercontent.com/GlueOps/gcp-cloudbuild-substitution-variables/main/gcsvh.sh
        | bash
    entrypoint: bash
  - name: node
    args:
      - install
      - '--only=production'
    entrypoint: npm
  - name: gcr.io/cloud-builders/gcloud
    args:
      - app
      - deploy
timeout: 300s
options:
  logging: STACKDRIVER_ONLY

```
