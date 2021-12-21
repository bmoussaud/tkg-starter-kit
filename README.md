# Tanzu Kubernetes Grid Starter Kit

This projects includes a set of Terraform scripts to provision a
[Tanzu Kubernetes Grid](https://tanzu.vmware.com/kubernetes-grid) cluster
with a bunch of core services:

- [Let's Encrypt](https://letsencrypt.org) certificate generation with [cert-manager](https://cert-manager.io)
- [Contour](https://projectcontour.io) (ingress controller)
- [External-DNS](https://github.com/kubernetes-sigs/external-dns) with [AWS Route 53](https://aws.amazon.com/route53/) integration (expose your Kubernetes services/ingresses as DNS records)

You can also deploy optional services:
- [Harbor](https://goharbor.io) (OCI-compatible registry)
- [Kubeapps](https://kubeapps.com) (a tool for deploying Helm charts)
- [Concourse](https://concourse-ci.org)
- [Jenkins](https://www.jenkins.io)

Using these Terraform scripts, you can deploy those services to your TKG cluster in a matter of minutes.

**Please note this project is not for production use!**

This implementation is designed to work with
[Tanzu Mission Control](https://tanzu.vmware.com/mission-control) managed clusters
using AWS as a provisioner.

## How to use it?

### Setting up your cluster

Create a TKG cluster from TMC. For best results, use at least 3 worker nodes.

As some of the services you're about to deploy use privileged containers,
you need to run this command for your cluster in order to enable this feature:

```bash
$ kubectl create clusterrolebinding privileged-cluster-role-binding --clusterrole=vmware-system-tmc-psp-privileged --group=system:authenticated
```

### Creating a Route 53 hosted zone

You need to create a public
[Route 53 hosted zone](https://console.aws.amazon.com/route53/v2/hostedzones#CreateHostedZone):
this zone will be used by cert-manager and External-DNS to register DNS records for your domain.

![Create a public hosted zone](images/aws-route53-hostedzone.png)

Then, make sure you configure your DNS registrar using the `NS` records from your Route 53 hosted zone.

## Configuring 

Create a [config-custom/myvalues.yml](config-custom/myvalues.yml) file based on the read-only file available in [config/values.yml](config/values.yml)

Edit this file accordingly:

```yaml
#@data/values
---
#! Default configuration values.
#! DO NOT EDIT THIS FILE: create a new file in config-custom instead.

#! Set base domain: all services are attached to this domain.
#! Please note that this domain must match a HostedZone in your AWS configuration.
DOMAIN: <put your domain here> 

#! Set AWS credentials.
AWS_ACCESS_KEY: xxxxxxx
AWS_SECRET_KEY: yyyyyy
AWS_REGION: eu-west-1

#! external-dns can create Route53 entries from your Ingress/Service objects.
#! Set to true to also monitor Contour HTTPProxy objects.
ENABLE_EXTERNAL_DNS_WITH_CONTOUR: true

#! Set default cluster issuer to use with Ingress objects.
#! 2 clusters issuers are available by default: letsencrypt-staging and letsencrypt-prod.
CERT_MANAGER_ISSUER: letsencrypt-staging

#! Set Let's Encrypt issuer email.
LETS_ENCRYPT_ISSUER_EMAIL: you@youremaildoman.org

#! Set password for Harbor user admin.
HARBOR_ADMIN_PASSWORD: mldmlfjkdqslmqkjg

#! Set password for Concourse user admin.
CONCOURSE_ADMIN_PASSWORD: changeme

#! Set password for Jenkins user admin.
JENKINS_ADMIN_PASSWORD: changeme
```

Get your AWS credentials from your AWS console
([My security credentials](https://console.aws.amazon.com/iam/home#/security_credentials)).
Create an access key, and record the secret key.

You're ready to start !

## Running

To install the default components (external-dns & cert-manager & lets-encrypt) just run

```bash
$ make init
```

```bash
$ make deploy-base
```

Additional components cant be install

```bash
$ make deploy-<component>
```

The available components are:
* contour
* harbor
* concourse
* jenkins
* kubeapps
* kpack
* artifactory
* prometheus

## Dependencies

* Makefile
* Terraform
* Kubectl
* Carvel Tools: ytt, kapp

Enjoy!

## Contribute

Contributions are always welcome!

Feel free to open issues & send PR.

## License

Copyright &copy; 2021 [VMware, Inc. or its affiliates](https://vmware.com).

This project is licensed under the [Apache Software License version 2.0](https://www.apache.org/licenses/LICENSE-2.0).
