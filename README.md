# Kata Admission controller webhook

Implement a simple admission controller webhook to annotate pods with the 
Kata runtime class.

## How to build the admission controller

First build the admission controller image and the associated
Kubernetes YAML files required to instantiate the admission
controller.

```bash
$ podman build -t katadocker/kata-webhook-example:latest -f Dockerfile
$ ./create_certs.sh
```

> **Note:**
> Image needs to be published for the webhook needs to work. Alternately
> on a single machine cluster change the `imagePullPolicy` to use the locally
> built image.

## Making Kata the default runtime using an admission controller

Today in `crio.conf` `runc` is the default runtime when a user does not specify
`runtimeClass` in the pod spec. If you want to run a cluster where Kata is used
by default, except for workloads we know for sure will not work with Kata, use
the [admission webhook](https://kubernetes.io/docs/reference/access-authn-authz/extensible-admission-controllers/#admission-webhooks)
and sample admission controller we created by running

```bash
$ kubectl apply -f deploy/
```

The webhook mutates pods to use the Kata runtime class for all pods except
those with 

* `hostNetwork: true` 
* namespace: `rook-ceph` and `rook-ceph-system`

### Advanced filtering

Add the -regex-matching-namespace parameter in deploy/webhook.yaml
followed by a regular expression, if this is set and pattern is valid
kata will be applied only on matching namespaces (the above excluded).
In case invalid regex pattern is set webhook pod will fail to start.
