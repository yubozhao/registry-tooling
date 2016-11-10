# Registry Tooling

Tools for working with Docker registries, especially those using self-signed
certificates.

Currently the tool will install a Docker registry on any Kubernetes cluster, secured with a self-signed certificate. It can also install the certificate into local Docker engines (e.g. developer's laptops).

Whilst there is an existing [cluster addon to start a
registry](https://github.com/kubernetes/kubernetes/tree/master/cluster/addons/registry),
it suffers from several flaws:

 - It does not use TLS. This means all transfers are unencrypted.
 - Each cluster node runs an instance of haproxy (the kube-registry-proxy image).
 - Another proxy has to be set-up to enable access from developer's machines

Using this tool will:

 - Install a registry on the current cluster with a self-signed certificate.
 - Configure all nodes to access the registry via TLS.
 - Optionally configure a local Docker daemon (e.g. developer's laptop) to
   access the registry via TLS.
 - Use NodePorts to avoid the need to run haproxy.

It will not currently configure a storage backend; please take a look at the
config files to see how to do this.

The script has been tested with
[minikube](https://github.com/kubernetes/minikube) and GCE clusters. 

WARNING: This will do funky stuff like edit /etc/hosts. It will warn before
doing this, but please be aware that it could break things. If you want to get a
secure registry running on existing cluster already handling load, I suggest you
look at what the scripts do and run the steps manually.


## Usage

### Installing a Registry on Kubernetes

The scripts will target whichever cluster `kubectl` currently points at.
Assuming your cluster is up-and-running, try:

```
$ ./secure-registry.sh -k
```

Once that completes, you should have running registry with certificates copied
to all nodes and networking configured. You can then configure the local Docker daemon to access the registry with:

```
$ sudo ./secure-registry.sh -c
```

This command should work on any Linux or Docker for Mac host whose kubectl is
pointing at a cluster running a configured registry. We can then test with:


```
$ docker pull redis
...
$ docker tag redis kube-registry.kube-system.svc.cluster.local:31000/redis
$ docker push kube-registry.kube-system.svc.cluster.local:31000/redis
...
$ kubectl run r1 --image kube-registry.kube-system.svc.cluster.local:31000/redis
```

Please note that it sometimes takes a few minutes for DNS to update.

## Minikube

If you're using minikube, note that you can also use the Docker daemon in the VM
to access the registry. Rather than using the tool to install a certificate with
`-c` just do:

```
$ eval $(minkube docker-env)
```

## Further Development

Was this useful to you? Or would you like to see different features? 

Container Solutions are currently looking at developing tooling for working with
images and registries on clusters. Please get in touch if you'd like to hear
more or discuss ideas.

 - adrian.mouat@container-solutions.com
 - [@adrianmouat](https://twitter.com/adrianmouat)

