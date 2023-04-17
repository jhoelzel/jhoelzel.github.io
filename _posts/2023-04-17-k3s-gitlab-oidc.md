---
layout: post
title: Kubernetes SSO with OIDC and GitLab in k3s
subtitle: How to secure your cluster the right way
categories: kubernetes
tags: [kubernetes, sso, k3s, oidc]
---

With the increasing adoption of microservices, the need for effective and **efficient** management of access and authentication has become paramount. This has led to the rise of Single Sign-On (SSO) solutions in Kubernetes environments, which not only provide a more secure approach, but also allow you to give users access to your cluster based on groups that you already have created for your ogranisation.

## What is OIDC?

OpenID Connect (OIDC): OIDC is a widely-accepted authentication protocol built on top of OAuth 2.0. It enables clients to verify the identity of users based on the authentication performed by an authorization server. OIDC provides additional features, such as single sign-on (SSO), which simplifies the user experience across multiple applications. OpenID Connect allows clients of all types, including Web-based, mobile, and JavaScript clients, to request and receive information about authenticated sessions and end-users as well as to obtain basic profile information about the End-User in an interoperable and REST-like manner. 

You can learn more about it on the [OpenID Website](https://openid.net/connect/)

## Integrating OIDC with GitLab and k3s

Integrating OIDC with K3s and Gitlab is very easy, simply follow the steps provided and you will have a demo system ready on your localhost.

### Step 1: Configure GitLab 

* Create a new GitLab application by navigating to "User Settings" > "Applications" > "New Application."
* Enter a name and redirect URI (e.g., "https://k3s.example.com/callback") for your application.
* Select the "openid" and "profile" scopes, then click "Save application."
* Note the "Application ID" and "Secret" for later use.
* Step 2: Configure k3s to Use GitLab as an OIDC Provider

### Step 2: Install the k3s server with the appropriate values from the previous step

K3s come with an integrated apiserver and much more, therefore including OIDC in the mix, requires you simply to pass along your application secrets from the previous step.

``` Console
curl -sfL https://get.k3s.io | sh -s - --kube-apiserver-arg="oidc-issuer-url=https://gitlab.example.com" --kube-apiserver-arg="oidc-client-id=<Application ID>" --kube-apiserver-arg="oidc-username-claim=email" --kube-apiserver-arg="oidc-username-prefix=oidc:" --kube-apiserver-arg="oidc-groups-claim=groups_direct" --kube-apiserver-arg="oidc-groups-prefix=gitlab:"
```

### Step 3: Configure Kubernetes RBAC

Since we are now giving groups access to our cluster and **not indivdual users**, we need to setup some RBAC rules in order to make sure that every group has the right permissions.
Create a new ClusterRole with appropriate permissions for your users:

```YAML
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: gitlab-oidc-user
rules:
- apiGroups: [""]
  resources: ["pods", "services", "configmaps", "secrets"]
  verbs: ["get", "list", "watch", "create", "update", "delete"]

```

Create a ClusterRoleBinding to map the GitLab OIDC users to the ClusterRole:

```YAML
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: gitlab-oidc-user-binding
subjects:
- kind: Group
  name: "gitlab:<your-gitlab-group>"
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: gitlab-oidc-user
  apiGroup: rbac.authorization.k8s.io
```

Or if you are brave ennough you can create a group called "admins" in gitlab and assign the following role-binding to assign admin rights to all members:

```YAML
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: oidc-cluster-admin
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: Group
  name: gitlab:admin
```

Replace "your-gitlab-group" with the appropriate GitLab group name that you want to grant access to your Kubernetes cluster.

### Step 4: Test Your Configuration

Obtain an OIDC token from GitLab using the following command, replacing <Application ID> and <Application Secret> with the values obtained in Step 1:

```Console
curl -s -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "grant_type=password&username=<your_gitlab_username>&password=<your_gitlab_password>&client_id=<Application ID>&client_secret=<Application Secret>&scope=openid" "https://gitlab.example.com/oauth/token" | jq -r '.id_token'
``` 

Save the obtained token to an environment variable:

```Console
export KUBECONFIG_OIDC_TOKEN=<obtained_token>
```

Set up a new kubeconfig context with OIDC authentication:
``` YAML
apiVersion: v1
kind: Config
current-context: k3s-oidc
clusters:
- cluster:
    server: https://k3s.example.com:6443
    certificate-authority-data: <k3s-ca-data>
  name: k3s-oidc
contexts:
- context:
    cluster: k3s-oidc
    user: gitlab-oidc
  name: k3s-oidc
users:
- name: gitlab-oidc
  user:
    token: ${KUBECONFIG_OIDC_TOKEN}
```

Replace <k3s-ca-data> with the contents of your k3s server's /etc/rancher/k3s/server/tls/server-ca.crt file, encoded in base64 format.

Save the kubeconfig to a file (e.g., oidc-kubeconfig.yaml) and set the KUBECONFIG environment variable:

```Console
export KUBECONFIG=oidc-kubeconfig.yaml
```

Test your configuration by running a simple kubectl command:
``` Console
kubectl get nodes
```

If everything is configured correctly, you should see a list of the nodes in your k3s cluster.

### kube-login is more efficient

[kube-login](https://github.com/int128/kubelogin) is a kubectl plugin designed for Kubernetes OpenID Connect (OIDC) authentication, which is commonly referred to as kubectl oidc-login.
Kubelogin is specifically developed to function as a client-go credential plugin. Upon executing kubectl, kubelogin triggers the browser to initiate a login to the designated provider. Once authenticated, kubelogin retrieves a token from the provider, enabling kubectl to access Kubernetes APIs.

After installing kube-login, all you have to do is to adapt your kubeconfig like this:

``` YAML
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: <certdata>
    server: https://<yourcluster>:6443
  name: default
contexts:
- context:
    cluster: default
    user: default
  name: default
current-context: default
kind: Config
preferences: {}
users:
- name: default
  user:
    auth-provider:
      config:
        client-id: <clientid>
        client-secret: <clientsecret>
        extra-scopes: email
        idp-issuer-url: https://gitlab.yourdomain.yourtld
      name: oidc
- name: oidc
  user:
    exec:
      apiVersion: client.authentication.k8s.io/v1beta1
      args:
      - oidc-login
      - get-token
      - --oidc-issuer-url=https://gitlab.yourdomain.yourtld
      - --oidc-client-id=<clientid>
      - --oidc-client-secret=<clientsecret>
      - --oidc-extra-scope=email
      command: kubectl
      env: null
      interactiveMode: IfAvailable
      provideClusterInfo: false
```

After that all you have to do is adapt your gitlab application redirection url accordingly with: "http://localhost:8000" and you should be all set!
  
If you need help setting up OIDC for your cluster or are interested in more sophisticated techniques to secure your cluster, feel free to contact me on linkein.
