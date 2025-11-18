# openshift-local-developer-env

I tried out the Local Development from Redhat Openshift. This Github Repo helps developers to setup an Openshift Environment on their local workstation.

## Openshift Installation Guide

Many organisations use [Openshift](https://access.redhat.com/documentation/en-us/red_hat_openshift_local/2.23/html-single/getting_started_guide/index) as "in-house" container management platform. Therefore, it can be handy to have a Openshift instance on your local workstation serving you as a [local dev environment](https://loft.sh/blog/the-definitive-guide-to-development-environments/#local-dev-environment?utm_medium=reader&utm_source=loft-blog&utm_campaign=blog_the-definitive-guide-to-development-environments) for your daily development tasks.

### Windows

- t.b.d.

### MacOS (ARM-based)

*Side Note: While the latest M-Series Macbook Pro Notebooks are Power Houses that can easily host heaps of running containers in the local Openshift, be aware that as of today still many Redhat Images do not support ARM-Based Processors.*

First [download](https://developers.redhat.com/content-gateway/rest/mirror/pub/openshift-v4/clients/crc/2.31.0) the install file and follow the [installation steps](https://www.redhat.com/sysadmin/install-openshift-local).


Start this in one terminal session
```sh
crc daemon
```

Verify the installation and give it some hardware resources (CPU, RAM, Storage): 
```sh
crc version
crc config view
crc config set consent-telemetry yes
crc config view
crc setup
crc config set cpus 8
# Give it some hardware space, 50 GB
crc config set disk-size 45600
crc config get disk-size
```

Start the thing
```sh
# Ensure dependencies are up-to-date
crc setup
```

If not already login to your Redhat Developer Account and go to [download page](https://developers.redhat.com/register?intcmp=701f20000012ngPAAQ) and download another pull-secret.
```
export PATH_TO_PERSONAL_PULL_SECRET=~/path/to/pull-secret.txt
```
_(Note: Change `~/Documents/secrets/pull-secret.txt` to wherever your stored yours)_


```
# Point it the downloaded secret that matches your installation binary
crc start -p $PATH_TO_PERSONAL_PULL_SECRET
```

```
oc get nodes
eval $(crc oc-env)
```

Remove stuff again
```
crc stop
crc cleanup
```

Use it! Create a demo app using the [sample provided by Redhat](https://www.redhat.com/sysadmin/build-deploy-application-openshift).


```sh
oc login -u developer https://api.crc.testing:6443
oc new-project hello-world
oc new-app https://github.com/rgerardi/hellogo.git
oc new-app https://github.com/rgerardi/hellogo.git
oc logs -f bc/hellogo
oc status
oc expose deploy hellogo --port 3000
oc expose service hellogo
oc get route hellogo --template '{{ .spec.host }}'

# Test it
curl http://hellogo-hello-world.apps-crc.testing
curl "http://$(oc get route hellogo --template '{{ .spec.host }}')"
oc get pod -l deployment=hellogo
```

## Refs
- [More in information on local Openshift](https://access.redhat.com/documentation/en-us/red_hat_openshift_local/2.5/html/getting_started_guide/introducing_gsg)


### Reinstall after license expiry

```
crc stop
crc cleanup
```
Go to [download page](https://developers.redhat.com/register?intcmp=701f20000012ngPAAQ) and download another pull-secret
```
crc start -p ~/Downloads/new-pull-secret.txt
```
Observe the new license:

![](./images/Screenshot%202025-09-22%20at%2015.52.25.png)


### Troubleshooting

- Marketplace does not show up
  Possible Solution: not authenticated against the registry
  ```sh
  oc create secret docker-registry redhat-registry --docker-server=registry.redhat.io --docker-username=<your-username> --docker-password=<your-password> --docker-email=<your-email>
  ```

  ```sh
  oc secrets link default redhat-registry --for=pull
  ```
  RESTART!
  ```sh
  crc stop
  crc start -p $PATH_TO_PERSONAL_PULL_SECRET
  ```
  
  Restart all the pods that are in crash-loop manually!

  ![]()


## Other Openshift Dependencies


You might also need
- [cert-manager-operator](https://docs.openshift.com/container-platform/4.10/security/cert_manager_operator/cert-manager-operator-install.html)


## Use it

```
brew install openshift-cli  
```
Details in the [medium Blog Post - install openshift cli using brew](https://ksummersill.medium.com/install-openshift-cli-using-brew-for-macos-and-connect-to-k8s-cluster-7f8efff0e5e)