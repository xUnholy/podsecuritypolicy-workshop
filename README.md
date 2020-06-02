# Pod Security Policies

A Pod Security Policy is a cluster-level resource that controls security sensitive aspects of the pod specification. The PodSecurityPolicy objects define a set of conditions that a pod must run with in order to be accepted into the system, as well as defaults for the related fields.

> Source: https://kubernetes.io/docs/concepts/policy/pod-security-policy/

# Scenarios

## Apply Privileged Deployment

Expectation: Failure

### Details

Copy the contents of the [Privileged Deployment](./deploy-privileged.yaml) and then run the following command in the cluster:

```bash
kubectl apply -f ./serviceaccount.yaml
kubectl apply -f ./deploy-privileged.yaml
```

This will create your deployment in the `example` namespace but the pod will not start up.

### Debug

The Pod hasn't started - Check either the Deployment or Replica Set for output:

```bash
kubectl get deploy -o yaml
```

```bash
kubectl get rs -o yaml
```

Error will look something like the following:

```bash
Error creating: pods "<podID>" is forbidden: unable to validate against any pod security policy: [spec.securityContext.hostNetwork: Invalid value: true: Host network is not allowed to be used spec.securityContext.hostPID: Invalid value: true: Host PID is not allowed to be used spec.containers[0].securityContext.privileged: Invalid value: true: Privileged containers are not allowed]
```

## Apply Non-Privileged Deployment

Expectation: Success

### Details

Copy the contents of the [non-Privileged Deployment](./deploy-non-privileged.yaml) and then run the following command in the cluster:

```bash
kubectl apply -f ./deploy-non-privileged.yaml
```

This will create your deployment in the `example` namespace but this time the pod will start up with no errors.

## Apply Privileged Deployment With PSP

Expectation : Success

### Details

To enable the privileged deployment to successfully run within the cluster we'll need to do the following:

1. Create PodSecurityPolicy Resource
2. Create ServiceAccount
3. Create ClusterRole
4. Create RoleBinding

To do this simply `kubectl apply -f <file>` for each of these items.

Validate the Service Account we've created indeed has been granted correct access to use the Pod Security Policy allowing privileged pods:

```bash
k auth can-i use podsecuritypolicy/psp.example --as=system:serviceaccount:example:example-psp-sa
```

The expected response is `yes`, however, if the response is `no` then there is an issue with potentially any of the following:

- ServiceAccount Name Doesn't Match
- PodSecurityPolicy Is Missing Specified Access
- Namespace Name in the command Doesn't Match the Actual Namespace
- RoleBinding Must Be Deployed In Same Namespace As Service Account

Assuming your result was `yes` we can apply the privileged deployment with the requested permissions:

Apply Privileged Deployment:

```bash
kubectl apply -f ./deploy-privileged.yaml
```

Congratulations!! Your Pod should be starting with the request permissions whilst using PodSecurityPolicies within the cluster.
