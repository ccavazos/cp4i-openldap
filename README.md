# OpenLDAP on OpenShift for IBM Cloud Pak for Integration

Installs OpenLDAP and phpLDAPadmin with a few initial users for the purposes of showing LDAP integration capabilities with Cloud Pak for Integration.

> Recommended to use this **only** for **non-production** purposes. Any organization will have some an LDAP server that can be leveraged to configure Cloud Pak for Integration.

## Tested on

> The installation of the Helm CLI is done directly from the OpenShift website. See below.

- OpenLDAP running with **HTTPS disabled**
- OpenShift Cluster **4.4.17** running on [IBM Cloud ROKS Service](https://www.ibm.com/cloud/openshift)
- Helm CLI **v3.2.3+4.el8**
- Cloud Pak for Integration 2020.2.1

## Installation

> Make sure you have already logged into the OpenShift CLI (i.e. `oc login`)

```bash
$ RELEASE_NAME=cp4i-openldap
$ NAMESPACE=cp4i-ldap
$ oc new-project $NAMESPACE
$ oc adm policy add-scc-to-user anyuid -z default -n $NAMESPACE
$ helm install $RELEASE_NAME https://github.com/ccavazos/cp4i-openldap/releases/download/0.1.7/cp4i-openldap-0.1.7.tgz --namespace $NAMESPACE
NAME: cp4i-openldap
LAST DEPLOYED: Thu Aug 27 18:20:27 2020
NAMESPACE: cp4i-ldap
STATUS: deployed
REVISION: 1
TEST SUITE: None
```

### Manage users via phpLDAPadmin

By default the LDAP UI is not exposed outside the cluster (and also is insecure) but you can port forward to access it. [Read more about port forward](https://blog.ccavazos.co/posts/openshift-cli-port-forwarding).

> In this example, `192.168.1.1` is the IP of your computer

```bash
$ oc port-forward $(oc get pods -n $NAMESPACE | grep $RELEASE_NAME-admin | awk '{print $1}') -n $NAMESPACE 8080:80 --address=192.168.1.1
Forwarding from 192.168.1.1:8080 -> 80
```

Once this is setup you can access it via the browser at `http://192.168.1.1:8080/`. The Login DN is `cn=admin,dc=ibm,dc=com` and the password is defined in the values of this chart.

### Manage users via OpenShift

To get the information of the users created check the config map

```bash
~ oc describe configmap cp4i-openldap-seedusers -n $NAMESPACE
```

If you do changes to the users (add, edit, remove) make sure you delete the pod

```bash
$ oc get pods
NAME                                   READY   STATUS    RESTARTS   AGE
cp4i-openldap-67676b9c5b-tft6x         1/1     Running   0          2m
cp4i-openldap-admin-5c98969977-rt2np   1/1     Running   0          2m

$ oc delete pod cp4i-openldap-67676b9c5b-tft6x -n $NAMESPACE
pod "cp4i-openldap-67676b9c5b-tft6x" deleted
```

The deployment will recreate the pod automatically and the changes will be loaded.

## Install Helm CLI

> Source: [OpenShift Docs](https://docs.openshift.com/container-platform/4.4/cli_reference/helm_cli/getting-started-with-helm-on-openshift-container-platform.html#installing-helm_getting-started-with-helm-on-openshift)

For macOS:

Download the CLI

```bash
~ curl -L https://mirror.openshift.com/pub/openshift-v4/clients/helm/latest/helm-darwin-amd64 -o /usr/local/bin/helm
```

Allow execution permissions

```bash
~ chmod +x /usr/local/bin/helm
```

Validate the installation

```bash
$ helm version
version.BuildInfo{Version:"v3.2.3+4.el8", GitCommit:"2160a65177049990d1b76efc67cb1a9fd21909b1", GitTreeState:"clean", GoVersion:"go1.13.4"}
```

## Assets

Kubernetes Assets in this chart

### OpenLDAP

[Official site](http://www.openldap.org/)

Default values:

```yaml
OpenLdap:
  OpenLdap:
  Image: "docker.io/osixia/openldap"
  ImageTag: "1.4.0"
  ImagePullPolicy: "Always"
  Component: "openldap"
  Replicas: 1
  Cpu: "512m"
  Memory: "200Mi"
  Organisation: "Example Inc."
  Domain: "ibm.com"
  AdminPassword: "Passw0rd!"
  SeedUsers: 
    usergroup: "cp4iusers"
    userlist: "user1,user2,user3,user4"
    initialPassword: "cp4iForAll"
```

### phpLDAPadmin


[Official site](http://phpldapadmin.sourceforge.net/)

Default values:

```yaml
PhpLdapAdmin:
  Component: "phpadmin"
  Image: "docker.io/osixia/phpldapadmin"
  ImageTag: "0.9.0"
  ImagePullPolicy: "Always"
  Replicas: 1
  Cpu: "512m"
  Memory: "200Mi"
  Https: "false"
```

## Credits

- The original version of [this project](https://github.com/ibm-cloud-architecture/icp-openldap) was inteded to be used for IBM Cloud Private.
- I was introduced to this repo by [iamjamilspain](https://github.com/iamjamilspain) who also contributed to the development of this updated version.
- The work that inspired the original project was done by the [Samsung Cloud Native Computing Team](https://github.com/samsung-cnct).
