# Hands-on Red Hat Developer Hub Lab

In this lab, you will go through the process of creating, building, and deploying a sample application using the services provided by the Red Hat Developer Hub.  This process will include:

* Creating a GitHub Repo for your application
* Provide a method to compile and build the application
* Building a Dockerfile to containerize your application image
* Uses Open Liberty Operator and tools to aid in deployment using standard GitOps patterns
* Utilize GitHub actions to trigger CI/CD process
* Utilize standard CI/CD tools like Tekton, ArgoCD and Kubernetes

# Steps:

- [Hands-on Red Hat Developer Hub Lab](#hands-on-red-hat-developer-hub-lab)
- [Steps:](#steps)
  - [1. Initial OCP setup](#1-initial-ocp-setup)
    - [Login to the OpenShift console, using the following URL:](#login-to-the-openshift-console-using-the-following-url)
    - [Install Red Hat Marketplace Operators](#install-red-hat-marketplace-operators)
    - [Use default project](#use-default-project)
  - [2. Create Developer Hub instance](#2-create-developer-hub-instance)
    - [Apply the config map for our Developer Hub instance](#apply-the-config-map-for-our-developer-hub-instance)
    - [Create secret for the backend Developer Hub](#create-secret-for-the-backend-developer-hub)
  - [3. Add GitHub integration](#3-add-github-integration)
    - [Create secrets for GitHub integration](#create-secrets-for-github-integration)
  - [4. Create Keycloak resources](#4-create-keycloak-resources)
  - [5. Create Service Account](#5-create-service-account)
    - [Assign Role Binding to our Service Account](#assign-role-binding-to-our-service-account)
    - [Update Secrets with Service Account token](#update-secrets-with-service-account-token)
  - [6. Create ArgoCD instance](#6-create-argocd-instance)
    - [Update config map with ArgoCD values](#update-config-map-with-argocd-values)
  - [7. Create a dynamic plug-in config map](#7-create-a-dynamic-plug-in-config-map)
    - [Red Hat Developer Hub Operator](#red-hat-developer-hub-operator)
    - [Open Backstage Developer Hub](#open-backstage-developer-hub)
  - [8. Navigate through Backstage UI](#8-navigate-through-backstage-ui)
    - [END OF NEW INSTRUCTS, START OF OLD](#end-of-new-instructs-start-of-old)
    - [Add the RHDH helm chart to your cluster](#add-the-rhdh-helm-chart-to-your-cluster)
  - [2. Create Helm release](#2-create-helm-release)
  - [3. Setup GitHub Authentication \& Integration](#3-setup-github-authentication--integration)
    - [Create a GitHub App](#create-a-github-app)
  - [4. Configure Kubernetes plugins](#4-configure-kubernetes-plugins)
    - [Identify your new ServiceAccount’s authentication token](#identify-your-new-serviceaccounts-authentication-token)
    - [Add the kubernetes definition to your app-config-rhdh](#add-the-kubernetes-definition-to-your-app-config-rhdh)
  - [5. Configure ArgoCD plugin](#5-configure-argocd-plugin)

## 1. Initial OCP setup

### Login to the OpenShift console, using the following URL:

```bash
https://console-openshift-console.apps.ocp.ibm.edu
```

Use username: `ocadmin`. Use the password specified in the Lab Guide.

### Install Red Hat Marketplace Operators

For convenience, all required Operators have already been installed on your OpenShift cluster. These include:

![installed-operators](images/installed-operators.png)

- Open Liberty - framework for developing cloud-native Java microservices
- Red Hat OpenShift GitOps - a Continuous Delivery platform based on Argo CD
- Keycloak Operator - used to securely authenticate to applications
- Red Hat Developer Hub Operator - framework for building and managing developer portals 

To verify these operators are installed:
- Switch to the Developer view.
- Open up the `Operators` menu item, and select `Installed Operators`.
- Ensure the `Project` filter at the top of the list is set to `All Projects`.

### Use default project

For convenience, we will be using the `default` project/namespace for this lab. This allows service routes to be hard coded in YAML files that you will be asked to apply. This should make editing of the files easier, or not required, which limits the chance for typos or missed changes.  

>**IMPORTANT**: Ensure the `default` project is selected when completing the remaining steps in this lab.  

## 2. Create Developer Hub instance

From the Developer view:
- Go to `+Add` and select `Operator Backed` from the `Developer Catalog` section.

![operator-backed](images/operator-backed.png)

- From the list of options, select `Red Hat Developer Hub`.
- Click the `Create` button.

![rhdh-create-button](images/rhdh-create-button.png)

For now, leave all the fields with their default values and click `Create`.

Please be patient, as this may take several minutes to complete.

Once instantiated, you will be able to view the instance by clicking the `Topology` view.

![developer-hub-instance-list](images/developer-hub-instance-list.png)

Click the graph icon in the upper right to get a graphical view.

![developer-hub-instance-graph](images/developer-hub-instance-graph.png)

### Apply the config map for our Developer Hub instance

Click the `Import YAML` button located at the top of the console.

![apply-yaml](images/apply-yaml.png)

Copy and paste the contents of the file [app-config-rhdh.yaml](https://github.com/rhagarty/techxchange-rhdh/blob/main/app-config-rhdh.yaml) into the YAML editor.

>**NOTE**: Use `CTRL-v` to paste.

Click `Create` to save the config file.

This will create a `Config Map` named `app-config-rhdh`.

![app-config-rhdh](images/app-config-rhdh.png)

This version of the config map is filled with default values that will need to be update as we advance through the rest of the lab.

The config map has a number of variable names that we will need to assign proper values to using `Secrets`. It also has URLs that will need to be modified as we create services and proper links are generated.

### Create secret for the backend Developer Hub

From the Developer view:
- Click on the `Secrets` menu item.
- From the Secrets list, click on the `Create` drop-down menu on the right, and select `Key/value secret`.

In the Key/value secret form, enter the following values:
- Set `Secret name` to `secrets-rhdh`
- Set `Key` to `BACKEND_SECRET`
- Set `Value` to `password`

Click `Create` to add the secret.

Note that `BACKEND_SECRET` is referenced in the config map.

## 3. Add GitHub integration

For this you will need a public GitHub account.

Open a new browser tab to your GitHub account and log in.

- Go to your `Settings` window, and then click on the `<> Developer Settings` menu item.
- Click on `GitHub Apps`.
- Click `New GitHub App`.

From the new GitHub app form:
- Enter any unique name for the app
- Enter any valid URL for the homepage - this value will not be used anywhere
- Leave `Callback URL` blank
- **[IMPORTANT]** Turn off `WebHook - Active`

For `Repository Permissions`, set the values to match the following:
- Actions: RW
- Administration RW
- Commit Statuses R
- Contents: RW
- Environments: RW
- Issues: RW
- Metadata: R
- Packages: RW
- Pull Requests: RW
- Secrets: RW
- Variables: RW
- Workflows: RW

For `Organization permissions`, set the values to match the following:
- Administration: RW
- Members: R
- Variables: RW

Click `Create GitHub App` to save.
  
Once created, you will get generated data concerning your app. Some of these values will need to be added to config map. Values needed include:

- Application ID
- Client ID
- Client Secret (click option to generate this)
- Private Key (button to generate a key is at the bottom of the page)
- Personal Access Token

**DO NOT CLOSE this tab**! You will need to copy/paste these values to complete the next step.

### Create secrets for GitHub integration

Go back to your OpenShift console, and from the Developer view:
- Click on the `Secrets` menu item.
- From the Secrets list, click on the `Create` drop-down menu on the right, and select `Key/value secret`.

In the Key/value secret form, enter the following values:
- Set `Secret name` to `rhdh-secrets-github-integration`
- Set `Key` to `RHDH_GITHUB_INTEGRATION_APP_CLIENT_ID`
- Set `Value` to the Client ID

Use the `+ Add key/value` to add another secret. Repeat this action to add the following secrets:

- Set `Key` to `RHDH_GITHUB_INTEGRATION_APP_CLIENT_SECRET` and `Value` to the Client Secret
- Set `Key` to `RHDH_GITHUB_INTEGRATION_APP_ID` and `Value` to the Application ID
- Set `Key` to `RHDH_GITHUB_INTEGRATION_APP_PRIVATE_KEY` and `Value` to the Private Key
- Set `Key` to `RHDH_GITHUB_INTEGRATION_PERSONAL_ACCESS_TOKEN` and `Value` to the Personal Access Token
  
## 4. Create Keycloak resources

Apply the 3 Keycloak YAML files located in https://github.com/rhagarty/techxchange-rhdh/tree/main/keycloak.

Using the `Import YAML` button located at the top of the console, import the files in the following order:

1. `keycloak-postgres.yaml` - creates a database for Keycloak to connect to.

Insure the pod is up and running before continuing.

![postgres-pod](images/postgres-pod.png)

2. `keycloak-instance.yaml` - the OpenID Connect user management provider. 
3. `keycloak-realm.yaml` - pre-configured Keycloak users and access.

From the `Administrator` view, you should be able to see both Keycloak pods running:

![keycloak-pods](images/keycloak-pods.png)

Applying the YAML files will also create Keycloak secrets, which contain usernames and passwords. View them in the `Developer` view under `Secrets`:

![keycloak-secrets](images/keycloak-secrets.png)

## 5. Create Service Account

To perform this step, you will need to be in the Administrative view.

Navigate to `User Management`, then click on `ServiceAccounts`.

From the Service Account panel, click on `Create ServiceAccount`.

In the YAML editor, change the `name` value to `rhdh-sa`.

Click `Create` to save the Service Account.

>**NOTE**: The creation of the Service Account will automatically generate an associated secret, which will be needed in a later step.

### Assign Role Binding to our Service Account

To perform this step, you will need to be in the Administrative view.

Navigate to `User Management`, then click on `RoleBindings`.

From the Role Bindings panel, click on `Create binding`.

In the `Create RoleBinding` form, set the following values:
- `Binding type` to `Cluster-wide role binding`
- `RoleBinding` name to `rhdh-sa-rb`
- `Role name` select `cluster-admin` (see note)
- `Subject` select `ServiceAccount`
- `Subject namespace` select `default`
- `Subject name` to `rhdh-sa`

>**NOTE:** Setting `Role name` to `cluster-admin` is not a best practice from a developers perspective. This would typically be set appropriately by an actual cluster administrator.

Click `Create` to save the Role Binding.

### Update Secrets with Service Account token

When you created your Service Account, an associated secret should have been auto-generated. To find the secret:

- From the Admistrator view, click on `Workloads` and then `Secrets`
- Identify the secret with the same prefix name as your Service Account, and is of the type `service-account-token`

![sa-token-secret](images/sa-token-secret.png)

Click on the secret to show details.

From the details panel, click on `Reveal values` to view the token.

![reveal-sa-token](images/reveal-sa-token.png)

Copy the token so that we can add it to an existing Secret.

Return to the list of Secrets and edit the secret `secrets-rhdh`.

Add a new `key/value` pair to the secret, and set:
- key = `SA_TOKEN`
- value = token

`SA_TOKEN` is referenced in our `app-config` map.

## 6. Create ArgoCD instance

From the Admistrator view, click on `Installed Operators` and then click on the `Red Hat Openshift GitOps` operator.

Click on the `ArgoCD` tab.

![argocd-option](images/argocd-option.png)

Click on the `Create ArgoCD` button.

Accept all the default values and click `Create` to save. This will create an ArgoCD instance with the name `argocd`.

As a result, multiple ArgoCD pods will be deployed in your project (may take a few minutes).

![argocd-pods](images/argocd-pods.png)

To determine the ArgoCD route, navigate to `Networking` and click on `Routes`.

![argocd-route](images/argocd-route.png)

Click on the route to open up the ArgoCD UI.

![argocd-ui](images/argocd-ui.png)

To get the `admin` password to log into the ArgoCD UI, navigate to the Developer view, then click on `Secrets`. Locate the secret named `argocd-cluster`.

![argocd-cluster](images/argocd-cluster.png)

Click on it to show details. The admin password is located at the bottom of the panel

Remember the ArgoCD route and admin password, as they will be needed in the next step.

### Update config map with ArgoCD values

In the config map, navigate down to the `argocd` section.

Update the `url` and `password`, replacing with route URL and admin password obtained in the last step.

## 7. Create a dynamic plug-in config map

<TODO This may be improved - waiting on fixes from Erica>

`dynamic-plugins-rhdh`

This enables the backstage plug-ins
May be replaced by Liberty plug-in features

### Red Hat Developer Hub Operator

`developer-hub-rhdh`
Update YAML to point to config maps (app config and dynamic) and ArgoCD
Also add secrets

### Open Backstage Developer Hub

Administrator view
`Networking` -> `Routes`
Click on `backstage-developer-hub` Location URL
Sign in
username/pwd will be what we set in keycloak

## 8. Navigate through Backstage UI

Create template for Liberty Getting Started app
Enter template URL - https://github.com/OpenLiberty/liberty-backstage-demo/blob/main/liberty-template/template.yaml
Click `Analyze`
Follow steps in template
Run through pipeline

Click `Catalog` to see it was added
Click on it to see deployment info (hopefully will include Liberty tab)




### END OF NEW INSTRUCTS, START OF OLD

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

Click Create (we will do a Helm upgrade to make changes later)

Once both pods are up, go to the Topology view and click the Open URL icon to get your RHDH instance URL (this is your baseURL)

![helm-chart-upgrade](images/helm-chart-upgrade.png)

Follow the steps in the Create the ConfigMap & Upgrade the Helm chart sections hereIn the field under Root Schema → global → Shorthand for Users that do not want to specify….. make sure it has the last part of your cluster URL (starting with apps.xxx i.e. apps.sandbox-m2.ll9k.p1.openshiftapps.com)

![helm-chart-upgrade-2](images/helm-chart-upgrade-2.png)

Under Root Schema → Backstage Chart Schema → Backstage Parameters → Extra App Configuration files to inline into command arguments add the following:
* configMapRef: app-config-rhdh
* filename: app-config-rhdh.yaml

## 3. Setup GitHub Authentication & Integration

In the Developer view, go to ConfigMaps, select the three buttons next to your app-config-rhdh configMap, and select edit ConfigMap

Modify your app-config-rhdh configMap by adding the following inside the data definition:

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

Click Save to update this ConfigMap

To setup the secrets needed within your cluster, you’ll need to create some of these values in your public GitHub account.

### Create a GitHub App

* Homepage URL: use your RHDH URL (ex: https://backstage-developer-hub-ebanda-dev.apps.gmarcy-backstage.cp.fyre.ibm.com/)

* Callback URL: [RHDH URL]/api/auth/github/handler/frame (ex: https://backstage-developer-hub-ebanda-dev.apps.gmarcy-backstage.cp.fyre.ibm.com/api/auth/github/handler/frame)

Under the webhook section, unselect the Active button

Select Create GitHub app to finish creating this

Once this has been created, go into your GitHub app and select the Generate a new client secret button

You will need to make a note of this because you won’t be able to see it again

NOTE: you’ll need the App ID, Client ID, and Client Secret later 

Next, you’ll need to generate a private key on your GitHub App

if you’re not already on the GitHub App view go to Settings > Developer settings > GitHub Apps and click the Edit button for the app you need a private key for

Scroll to the bottom of the app page and click the Generate a private key button and save the .pem file

you will import this file later when creating the secrets-rhdh yaml file

Install your GitHub App

https://docs.github.com/en/apps/using-github-apps/installing-your-own-github-app

Next, you’ll need to create a personal access token in GitHub

https://github.ibm.com/settings/tokens/new

make sure repo, workflow, and delete_repo are selected 

You will need to make a note of this because you won’t be able to see it again

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

Create a ServiceAccount for Kubernetes 

In the Administrator view go to User Management → ServiceAccounts (make sure you’re in your project) and click Create ServiceAccount

![service-accounts](images/service-accounts.png)

Change the default name to something that makes sense to you (ex: kubernetes) and click Create

![service-accounts-create](images/service-accounts-create.png)

Create a new ClusterRoleBinding that gives your new ServiceAccount appropriate access

Click the + sign on the top right of the Open Shift console

![user-name](images/user-name.png)

In the YAML window paste the following: 

* replace the name under metadata with whatever you want to call this role binding 
* replace the name and namespace under the subjects section with your info

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
Click Create

### Identify your new ServiceAccount’s authentication token

In Administrator view navigate to Workloads → Secrets and search for a token that starts with the name of your ServiceAccount and is of type kubernetes.io/service-account-token ex: My service account is called “kubernetes”, so I searched for “kubernetes-token”

![secrets-k8s](images/secrets-k8s.png)

Open the secret and scroll to the bottom to find the token and click the copy button 

![secrets-token](images/secrets-token.png)

This will be the value for your K8S_SERVICE_ACCOUNT_TOKEN key.

Note: to get your kubernetes variables you will need to use CLI commands. You do this using your local terminal and log into your OpenShift cluster.

Setting up your local terminal:

If you do not already have oc (the OpenShift Command Line Interface (CLI)) installed, click the question mark in the top right hand corner menu of your OpenShift cluster and select Command Line Tools. Here you can download the appropriate oc for your machine.

Then click your email in the top right hand corner and select the Copy login command option, then IBM id, login and then select Display Token. This will give you what you need to log in to your cluster on your local terminal.

Identify your main Kubernetes variables:
* K8S_CLUSTER_NAME 
  * run oc config view --minify=true and look for the name definition under the cluster section 
* K8S_CLUSTER_URL
  * run oc config view --minify=true and look for the server definition under the cluster section
* K8S_CLUSTER_TOKEN
  * run oc whoami -t 

### Add the kubernetes definition to your app-config-rhdh

In the Developer view, navigate to ConfigMaps and find your app-config-rhdh

Switch to the YAML view an add the following section under the data definition and at the same level as the app definition 

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

be sure to save your changes 

Add the Kubernetes variables as environment variables to your developer hub deployment

In Administrator view navigate to Workloads → Deployments,  select the developer-hub instance, and click the Environment tab

![developer-hub-instance](images/developer-hub-instance.png)

Add your Kubernetes variables 

![k8s-variables](images/k8s-variables.png)

be sure to save your changes

Enable the Kubernetes dynamic plugins and run a helm upgrade

In the Developer view, navigate to Helm and find your developer-hub release

Click the three dots and click Upgrade

![upgrade-helm-version](images/upgrade-helm-version.png)

Go to Root Schema → global → Dynamic plugins configuration → List of dynamic plugins that should be installed in the backstage application

![dynamic-plugins](images/dynamic-plugins.png)

Click the Add List of dynamic plugins that should be installed in the backstage application link to add the following dynamic plugins 

`./dynamic-plugins/dist/backstage-plugin-kubernetes-backend-dynamic`

NOTE: The kubernetes backend plugin needs to be added before the other one

`./dynamic-plugins/dist/backstage-plugin-kubernetes`

You will need to included the integrity for this one

To get the integrity run npm view `@backstage/plugin-kubernetes` in your terminal

![view-plugins](images/view-plugins.png)

use the integrity value defined in the command output, for example:

`.integrity: sha512-mH36NXQ2+usTGDAo3RxeB1rG6YeaT2A8kWrJpJ75gQg8ChUMnhcF2a6dvZnNDMytl1KNiha8YSMapOwR1G5BiQ==`

## 5. Configure ArgoCD plugin

Install the Red Hat OpenShift GitOps and Red Hat OpenShift Pipelines operators 

In Administrator view, navigate to Operators → OperatorHub

search for the operator and install with the defaults

![operator-hub](images/operator-hub.png)

Create your own ArgoCD instance in your namespace(project)

Follow steps 1-9 from here: https://docs.openshift.com/gitops/1.12/argocd_instance/setting-up-argocd-instance.html

DURING STEP 5 ALSO DO THE FOLLOWING

In the server section find the insecure checkbox and enable it 

![argo-cd-instance](images/argo-cd-instance.png)

Identify your argocd variables

argocd instance name

In the Administrator view, navigate to Operators → Installed Operators and click on Red Hat OpenShift GitOps

In the Red Hat OpenShift GitOps window select the ArgoCD tab 

You should see your instance(s) along with the openshift-gitops instance 

![argo-cd-variables](images/argo-cd-variables.png)

In my case, my argocd instance name is: ebanda-argocd 

argocd instance url (ex: https://ebanda-argocd-server-ebanda-dev.apps.662f97af563fb9001ea854fa.cloud.techzone.ibm.com)

In the Administrator view, navigate to Networking → Routes and select your argocd server  In my case, my server is called ebanda-argocd-server

![argo-cd-server](images/argo-cd-server.png)

copy the Location of your argocd server

username (admin is the default value for all instances)

password (ex: HR7xNe28qVawWuScCstIib0jXGP6DOEn)

find the secret named [your-argocd-instance-name]-cluster (ex: ebanda-argocd-cluster)

In Administrator view navigate to Workloads → Secrets and select your cluster 

In the cluster view click the copy button to copy the admin.password

![argo-cd-secrets](images/argo-cd-secrets.png)

token: (EX: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJhcmdvY2QiLCJzdWIiOiJhZG1pbjpsb2dpbiIsImV4cCI6MTcxNDUyNTM1MiwibmJmIjoxNzE0NDM4OTUyLCJpYXQiOjE3MTQ0Mzg5NTIsImp0aSI6IjJhNjQ5NjgxLWU0MmItNDFjNi1iNDFmLWUzZjFhMGQzZWQwYiJ9.9cjszhHoyhcfzQBF6bY-pDA7YMm5LjS4A2hzrsda-a8)

in your browser(i.e. chrome, firefox, etc) open the developer tools(usually fn + F12) 

navigate to argocd-instance-url/sessions (ex: https://ebanda-argocd-server-ebanda-dev.apps.6629090f7a54bb001e244eca.cloud.techzone.ibm.com/sessions)

In the developer tools window, switch to the Network tab and note the /session request

in the Header section, look for the row called cookies, and then find the value called argocd.token and make a note of this value

![argo-cd-token](images/argo-cd-token.png)

Add the following ArgoCD definitions to your app-config-rhdh

argocd definition that points to your argocd instance (in the same indentation as app under data)

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

replace the instance name vale with your argocd instance name

replace the instance url value with your argocd instance url

Add a proxy definition to specify the argocd instance API in your app-config-rhdh as well. (This should also be at the same indentation as app under data)

replace the clusterRouterBase with your openshift console cluster base (ex: if this is your URL - https://console-openshift-console.apps.662f97af563fb9001ea854fa.cloud.techzone.ibm.com/ - then you’d set it to apps.662f97af563fb9001ea854fa.cloud.techzone.ibm.com)

modify the target url to reflect your argocd instance

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

Add the ArgoCD variables as environment variables to your developer hub deployment

In Administrator view navigate to Workloads → Deployments,  select the developer-hub instance, and click the Environment tab

Add the following keys and their values

* ARGOCD_USERNAME
* ARGOCD_PASSWORD
* ARGOCD_AUTH_TOKEN

Enable the ArgoCD plugins using a helm upgrade

In the Developer view, navigate to Helm and find your developer-hub release

Click the three dots and click Upgrade

![upgrade-helm-version-2](images/upgrade-helm-version-2.png)

Go to Root Schema → global → Dynamic plugins configuration → List of dynamic plugins that should be installed in the backstage application

![dynamic-plugins-2](images/dynamic-plugins-2.png)

Click the Add List of dynamic plugins that should be installed in the backstage application link to add the following dynamic plugins 

* ./dynamic-plugins/dist/roadiehq-backstage-plugin-argo-cd-backend-dynamic
* ./dynamic-plugins/dist/roadiehq-backstage-plugin-argo-cd
* ./dynamic-plugins/dist/roadiehq-scaffolder-backend-argocd-dynamic

![dynamic-plugins-3](images/dynamic-plugins-3.png)

You don’t have to specify the integrity for these 

Click the Upgrade button


