# Hands-on Red Hat Deveoper Hub Lab

In this lab, you will go through the process of creating, building, and deploying a sample application using the services provided by the Red Hat Developer Hub.  This process will include:

* Creating a GitHub Repo for your application
* Provide a method to compile and build the application
* Building a Dockerfile to containerize your application image
* Uses Open Liberty Operator and tools to aid in deployment using standard GitOps patterns
* Utilize GitHub actions to trigger CI/CD process
* Utilize standard CI/CD tools like Tekton, ArgoCD and Kubernetes

# Steps:

- [Hands-on Red Hat Deveoper Hub Lab](#hands-on-red-hat-deveoper-hub-lab)
- [Steps:](#steps)
  - [1. Initial OCP setup](#1-initial-ocp-setup)
    - [Create your own project](#create-your-own-project)
    - [Add the RHDH helm chart to your cluster](#add-the-rhdh-helm-chart-to-your-cluster)
  - [2. Create Helm release](#2-create-helm-release)
  - [3. Setup GitHub Authentication \& Integration](#3-setup-github-authentication--integration)
  - [4. Configure Kubernetes plugins](#4-configure-kubernetes-plugins)
  - [5. Configure ArgoCD plugin](#5-configure-argocd-plugin)

## 1. Initial OCP setup

Login to the OpenShift console using the following credentials:

- Account:
- Password:

### Create your own project

- Switch from the Administrator view to the Developer view
- Go to `+Add` and select `create a Project`
- In the pop-up that appears, enter a name for this project (e.g. name-dev) and select `Create`

### Add the RHDH helm chart to your cluster

In the Developer view go to +Add > Helm Chart:

![helm-chart](images/helm-chart.png)

From the Helm Charts panel, search for "Red Hat Developer Hub”:

![helm-chart-options](images/helm-chart-options.png)

Click the RedHat Developer Hub chart and click Create:

> **NOTE**: There are two available helm charts for RHDH, one from the Community source that defaults to developer-hub, and the other from the Red Hat source that defaults to redhat-developer-hub for the instance name. Use the one from the Community source that defaults to developer-hub.

![helm-rhdh](images/helm-rhdh.png)

Switch the Form view:

![helm-form](images/helm-form.png)

Click on Root Schema → global, and make sure that the Shorthand for Users that do not want to specify….. field has the last part of your cluster URL (starting with apps.xxx i.e. apps.sandbox-m2.ll9k.p1.openshiftapps.com)

It may have apps.example.com by default, just replace it with your cluster name - e.g. apps.662e4cddb86f79001e970405.cloud.techzone.ibm.com 

This can be found in the URL of your cluster.

NOTE: Your release name may be different depending on which RHDH helm chart you chose (ex: developer-hub vs. redhat-developer-hub)

## 2. Create Helm release

Click on "Create" button to start the process.

![helm-chart-create](images/helm-chart-create.png)

![helm-chart-upgrade](images/helm-chart-upgrade.png)

![helm-chart-upgrade-2](images/helm-chart-upgrade-2.png)

## 3. Setup GitHub Authentication & Integration

```bash
auth:
  allowGuestAccess: true
  environment: development
  providers:
    github:
      development:
        appId: ${GITHUB_APP_APP_ID}
        clientId: ${GITHUB_APP_CLIENT_ID}
        clientSecret: ${GITHUB_APP_CLIENT_SECRET}
        webhookUrl: ${GITHUB_APP_WEBHOOK_URL}
        webhookSecret: ${GITHUB_APP_WEBHOOK_SECRET}
        privateKey: |
            ${GITHUB_APP_PRIVATE_KEY}
enabled:
  github: true
integrations:
  github:
    - host: github.com
      token: ${GITHUB_APP_ACCESS_TOKEN}
```

Now navigate to the Secrets view of your cluster and select Create and from the drop down list select key/value secret

* Create a key/value Secret file named secrets-rhdh with the following keys (you’ll need to use the Add key/value button to add each of these individually)

  * Key: GITHUB_APP_ACCESS_TOKEN
    * Value: (This is the access token you just generated)

  * Key: GITHUB_APP_APP_ID
    * Value: (In your GitHub profile, under Developer Settings and GitHub apps, select your newly created app and at the top of this you’ll find a numerical value for your App ID)

  * Key: GITHUB_APP_CLIENT_ID
    * Value: (The client ID can be found underneath the App ID value in GitHub)

  * Key: GITHUB_APP_CLIENT_SECRET
    * Value: (This is the client secret you generated from your GitHub app that you should have noted down)

  * Key: GITHUB_APP_PRIVATE_KEY
    * Value: (This is the .pem file you downloaded, upload this)

  * Key: GITHUB_APP_WEBHOOK_SECRET
    * Value: (Set this to none when creating your secrets-rhdh)

  * Key: GITHUB_APP_WEBHOOK_URL
    * Value: (Set this to none when creating your secrets-rhdh)

Once you’ve inputted all of these keys and their values, select the Create button to create your secret file.

Navigate to the Helm tab and select Upgrade

Under Root Schema → Backstage chart Schema → Backstage parameters → Backstage container environment variables from existing Secrets, add secrets-rhdh as value

![add-secrets](images/add-secrets.png)

## 4. Configure Kubernetes plugins

![service-accounts](images/service-accounts.png)

![service-accounts-create](images/service-accounts-create.png)

![user-name](images/user-name.png)

```bash
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: kubernetes-cluster-admin
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: kubernetes
  namespace: ebanda-dev
```

![secrets-k8s](images/secrets-k8s.png)

![secrets-token](images/secrets-token.png)

```bash
kubernetes:
  serviceLocatorMethod:
    type: multiTenant
  clusterLocatorMethods:
    - type: config
      clusters:
        - url: ${K8S_CLUSTER_URL}
          name: ${K8S_CLUSTER_NAME}
          authProvider: serviceAccount
          skipTLSVerify: true
          skipMetricsLookup: true
          serviceAccountToken: ${K8S_SERVICE_ACCOUNT_TOKEN}
```

![config-map-yaml](images/config-map-yaml.png)

![developer-hub-instance](images/developer-hub-instance.png)

![k8s-variables](images/k8s-variables.png)

![upgrade-helm-version](images/upgrade-helm-version.png)

![dynamic-plugins](images/dynamic-plugins.png)

![view-plugins](images/view-plugins.png)

## 5. Configure ArgoCD plugin

![operator-hub](images/operator-hub.png)

![argo-cd-instance](images/argo-cd-instance.png)

![argo-cd-variables](images/argo-cd-variables.png)

![argo-cd-server](images/argo-cd-server.png)

![argo-cd-secrets](images/argo-cd-secrets.png)

![argo-cd-token](images/argo-cd-token.png)

```bash
argocd:
      username: ${ARGOCD_USERNAME}
      password: ${ARGOCD_PASSWORD}
      appLocatorMethods:
        - type: 'config'
          instances:
            - name: ebanda-argocd
              url: "https://ebanda-argocd-server-ebanda-dev.apps.662f97af563fb9001ea854fa.cloud.techzone.ibm.com"
              token: ${ARGOCD_AUTH_TOKEN}
```

```bash
clusterRouterBase: apps.6629090f7a54bb001e244eca.cloud.techzone.ibm.com
    proxy:
      endpoints:
        '/argocd/api':
          # url to the api of your hosted argoCD instance
          target: https://ebanda-argocd-server-ebanda-dev.apps.662f97af563fb9001ea854fa.cloud.techzone.ibm.com/api/v1/
          changeOrigin: true
          # this line is required if your hosted argoCD instance has self-signed certificate
          secure: false
          headers:
            Cookie:
              $env: ARGOCD_AUTH_TOKEN
```

![upgrade-helm-version-2](images/upgrade-helm-version-2.png)

![dynamic-plugins-2](images/dynamic-plugins-2.png)

![dynamic-plugins-3](images/dynamic-plugins-3.png)



