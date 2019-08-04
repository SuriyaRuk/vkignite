# vkignite

## Prepare K8s

How to run Kubernetes with Ignite support:

`$ docker-compose -f vkignite.yaml up`

Wait until `kubeconfig.yaml` is written into the current dir.

`$ export KUBECONFIG=$PWD/kubeconfig.yaml`

Run get nodes and you'll see:

```
$ kubectl get nodes
NAME     STATUS   ROLES   AGE     VERSION
ignite   Ready    agent   5h12m   v1.14.3-ignite-f5516fe-dev
```

## Define a Virtual Machine
Apply the Custom Resource Definition to Kubernetes to allow VM creation via `kubectl`.

```
$ kubectl apply -f crd/vm_crd.yaml
```
Then open an editor to define a VM with the above CRD.

```
apiVersion: chanwit.github.com/v1alpha1
kind: VirtualMachine
metadata:
  name: my-vm
spec:
  kernel: weaveworks/ignite-kernel:4.19.47
  image: weaveworks/ignite-ubuntu:latest
  cpus: 1
  memory: 512M
  diskSize: 1GB
```
save it as `examples/my-vm.yaml`.
The `VirtualMachine` CRD is the simplified counter-part of Ignite's `v1alpha1` config.

## Provision the VM

We then could be able to create a VM by:
```
$ kubectl apply -f examples/my-vm.yaml
```

To see that the VM is already provisioned:
```
$ kubectl get vm,pod
NAME                                      KERNEL                             IMAGE                             CPUS   MEMORY   SIZE   STATE
virtualmachine.chanwit.github.com/my-vm   weaveworks/ignite-kernel:4.19.47   weaveworks/ignite-ubuntu:latest   1      512M     1GB    Pending

NAME        READY   STATUS    RESTARTS   AGE
pod/my-vm   1/1     Running   0          2m3s
```

Then you now will be able to use `sudo ignite ssh default__my-vm` to get into the VM.
The VM name is in the `{namespace}__{name}` pattern.

To delete the VM,
```
$ kubectl delete vm my-vm
$ kubectl delete pod my-vm
```
