# Kubernetes Deployment Template

This projects demonstrates deploying a Node.js api to kubernetes utilizing Istio service mesh for A/B testing, sealedSecrets for publishing encrypted secrets to version control and RBAC.

## Scenario

We have a [Node.js](https://nodejs.org) application that collects and store Geospatial data.

There are two versions, **v1** and **v2**. Version v2 needs to be deployed and tested against Version v1 through **A/B** testing.

We also need to restrict access to secrets to only authorized users.

## Prerequisites
 1. A running kubernetes cluster or managed kubernetes cluster. This can be on AWS EKS, [GKE](https://cloud.google.com/kubernetes-engine) or any other cloud provider.
 2. Installed Istio service mesh. Install it **[here](https://istio.io/latest/docs/setup/getting-started/)**.
 3. Bitnami Sealed Secrets. Install this **[here](https://github.com/bitnami-labs/sealed-secrets?tab=readme-ov-file#installation)**.
 4. Apply registry credentials as a secret. In this case it is named `regcred`

## Walkthrough

 1. Apply the namespaces. The `istio-injection: enabled` label allows Istio to deploy sidecars besides pods that handle networking and monitoring.
 2. Encrypt your secrets with `kubeseal`. This will produce a file which will contain your sealed secrets which can be commited to version control safely. You can also apply the sealed secrets a you would with secrets.
 3. Apply the node.js deployments. Replace `IMAGE_REGISTRY_URL` with the repository url for your docker image.
 4. Apply the service.
 5. Apply the role. The role ***pods-admin*** grants all access to pods in the dev namespace while ***secrets-reader*** allows access to secrets in the dev namespace.
 6. Apply the role binding. The role binding attaches roles to users.
 7. Apply the [Istio gateways](https://istio.io/latest/docs/reference/config/networking/gateway/).
 8. Apply DestinationRule. A DestinationRule defines policies that apply to traffic intended for a service after routing has occurred. In this case we use this for spliting traffic between v1 and v2 by defining subsets for each deployment.
 9. Apply Virtual Service. The virtual service allows us to implement A/B testing, Canary releases, fault injection and other features. For more details check out the [docs](https://istio.io/latest/docs/reference/config/networking/virtual-service/). In this case we want to distribute traffic between **v1** and **v2** (A/B testing). We do this with the `weight` directive. 70% of incoming traffic is sent to v1 while 30% is sent to v2.


## Improvements

For further fine-grained traffic control, you can implement Egress Gateways to control outbound traffic. A scenario where this may be necessary is if you need egress traffic to go through a specific dedicated node for closer monitoring.

#### Real world example
Imagine a situation whereby a company's product is an API (eg. Payments) and you need your users to whitelist specific IP addresses to prevent unauthorized requests from malicious actors.