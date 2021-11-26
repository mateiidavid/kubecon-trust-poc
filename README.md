# KubeCon EU '22 POC: mTLS across boundaries using Linkerd, cert-manager and trust

**Problem statement**: managing certificates in an enterprise environment can
be a daunting task, especially when migrating to Kubernetes, or when switching
from an enterprise PKI. Typically, you will need different types of
certificates, depending on how you plan to secure your cluster; as a minimal
set-up, you will likely need to terminate TLS at an ingress level and ensure
you have the right certificates for your workloads to talk to the Kubernetes
apiserver. The problem is further accentuated when you want to lock down your
cluster using something like mTLS -- suddenly, you have a multitude of
certificates to manage -- identity management becomes non-trivial to support.

**Solution**: use open source software to bootstrap identity and mTLS for all
of your components!

## Workshop contents and feasibility

**General idea**: show how you can pull in a CA from an external PKI (e.g
`vault`) and use that to issue other certificates in a cluster. (@matei)
ideally we would start small here and show how everything would work together
step-by-step.

__(@matei)__: I think something that would be feasible to do would be to get an
issuer CA from vault and use that to sign two other CAs: __one for
bootstrapping ingress TLS certificate__ and __another for Linkerd (trust
anchor)__. We could potentially also get a third CA for k8s api-server (or for
Linkerd's webhooks). How I see this going in practice, this is mostly for Linkerd:

* Pull in certificate from `vault` using a `Vault` issuer type.
* Use issuer to create a Linkerd trust anchor (root CA); create an issuer for
  this root CA so we can sign intermediate issuers.
* Create an issuer certificate (and cert-manager issuer type) scoped to the Linkerd namespace.
* Use `trust` to distribute the root CA as a configmap in the Linkerd namespace (needed for installation).
* Install Linkerd and point the control plane to the root trust configmap and issuer certificate bundle (k8s `Secret`).

If we use the steps as outlined above, we'll be able to demonstrate how pulling
a certificate from outside the cluster (from Vault) we can install Linkerd
without needing any manual intervention; after the initial set-up, upgrades and
installs can be entirely automated (you don't need to cat/grep/awk or whatever
else to get the root or issuer passed in to the control plane). We can follow
the same steps for generating an ingress cert, at the end, we can double down
on everything and prove mTLS from ingress all the way to the server, all done
in an automated way.

### Questions
---

__(@matei)__: would this be what we have in mind? As far as feasibility goes, I
tested `trust` with Linkerd and it works well, but is this in line with what we
want the workshop to be about and show?

__(@matei)__: how else can we use `trust`? Are there any other cool things we
can do that I'm missing?



