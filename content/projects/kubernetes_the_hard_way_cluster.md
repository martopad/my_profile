+++
title = 'My_kubernetes_the_hard_way_cluster'
date = 2024-08-11T16:47:50+02:00
toc = true
+++

## Goals
1. Create my own Kubernetes cluster-- the hard way.
2. Use GlusterFS as my network storage.
3. Use Ansible to documenet and automate provisioning. 

## Setup
1. 1x Loadbalancer node
2. 2x Control Plane nodes
3. 2x etcd nodes
4. The rest, worker nodes

## Journey
As expected, I made this journey a lot harder for myself. So, instead of focusing
only on learning Kubernetes by using something like Minikube or K3s, I spent a
significant portion of my time learning how to provision a Kubernetes cluster from
scratch– it was worth it, though.

My journey was not without its setbacks. Initially, I followed a
[popular tutorial](https://github.com/kelseyhightower/kubernetes-the-hard-way),
only to be met with inconsistent behaviors when attempting to apply the lessons to
provision coreDNS, CNI, and other components. It was then that I discovered
[a fork](https://github.com/mmumshad/kubernetes-the-hard-way) that shed light on the
issue. The original kubernetes-the-hard-way was designed for GCP deployment,
explaining why the documentation and tutorials I followed didn't yield the expected results.

Sad as it is, I had to scrap days of work to follow Mmumshad's tutorial instead.
After following it, with changes due to my OS of choice, my Kubernetes cluster is finally working!

## Learnings

1. I can see why Ansible picked up popularity. Its syntax is friendly to non-developers.
Though coming from a developer background, I encountered some scenarios wherein I said
this would have been easier if I could code it directly. But, more often than not,
I was able to do what I wanted to do with Ansible. It also helped that Ansible is widely supported;
it even has a Portage module!
    * One feature that I appreciated was being able to detect and create 
    *If statements* depending on architecture. In this example, since my
    nodes are a mix between amd64 and arm64, the GlusterFS ebuild is masked for arm64,
    and I needed to do some portage configuration to unmask it.

```yaml
- name: Add accept keywords for GlusterFS for aarch64 (raspi5)
      ansible.builtin.copy:
        src: "{{ playbook_dir }}/../../config/static/root/etc/portage/package.accept_keywords/glusterfs"
        dest: "/etc/portage/package.accept_keywords/glusterfs"
      when:
        - ansible_architecture == "aarch64"
```
2. Since Gentoo is highly configurable, creating your ebuilds and app recipes are encouraged.
It makes sense since there are so many Linux apps. Though Kubernetes modules work well in Gentoo,
ipvsadm is not yet supported on arm64.
    * Though minor in this example, I had to create my own
    [Gentoo Portage overlay](https://github.com/martopad/gentoo-overlay) to install it.
    * etcd is also not supported for arm64 on Gentoo yet. But in that case, I
    just installed it on amd64 nodes. I will migrate it in the future.

3. GlusterFS is surprisingly easy to configure. Once you identify the Volume type you want, it is
just a matter of determining which drives you want to use as GlusterFS bricks in each node.

4. Kubernetes removed support for GlusterFS. Though not pretty, I resorted to having worker nodes
mount the Gluster volume and then exposing that as a mount point to containers.
    * Online threads are saying Kadalu is a suitable replacement. But it could be more straightforward
    to implement compared to what I did. I will likely use Kubernetes NFS support to mount Gluster volumes.

5. Though doing it the hard way consumed a lot of time, Kubernetes' declarative nature makes it a
delight to use once it is set. Although it still triggers my `this-is-too-easy-I-must-be-doing-something-wrong`
senses XD.
    * I used bare Kubernetes manifests to deploy stuff and it was still easy. Helm
    is on my sights for sure.
    * My Kubernetes Stack:
        1. Flannel for CNI
        2. CoreDNS
        3. Istio for ingress gateway
        4. Metallb for loadbalancing

## Results

Getting my own baremetal Kubernetes cluster up and running feels good!

Here are the nodes:
```bash
$ kubectl get -o wide nodes
NAME       STATUS   ROLES    AGE   VERSION   INTERNAL-IP    EXTERNAL-IP   OS-IMAGE       KERNEL-VERSION       CONTAINER-RUNTIME
rpi503     Ready    <none>   24d   v1.30.2   192.168.1.30   <none>        Gentoo Linux   6.6.31-v8-16k+       containerd://1.7.15
rpi504     Ready    <none>   24d   v1.30.2   192.168.1.32   <none>        Gentoo Linux   6.6.36-v8-16k+       containerd://1.7.15
rpi505     Ready    <none>   24d   v1.30.2   192.168.1.33   <none>        Gentoo Linux   6.6.36-v8-16k+       containerd://1.7.15
un100d02   Ready    <none>   28d   v1.30.2   192.168.1.34   <none>        Gentoo Linux   6.6.35-gentoo-dist   containerd://1.7.15
un100d03   Ready    <none>   28d   v1.30.2   192.168.1.35   <none>        Gentoo Linux   6.6.35-gentoo-dist   containerd://1.7.15
```

HA etcd node setup:
```bash
$ ETCDCTL_API=3 etcdctl endpoint --cluster status  --cacert=/etc/etcd/ca.crt --cert=/etc/etcd/etcd-server.crt --write-out=table --key=/etc/etcd/etcd-server.key
+---------------------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
|         ENDPOINT          |        ID        | VERSION | DB SIZE | IS LEADER | IS LEARNER | RAFT TERM | RAFT INDEX | RAFT APPLIED INDEX | ERRORS |
+---------------------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
| https://192.168.1.22:2379 | 2832ca0e95cb6fb0 |  3.5.12 |  7.7 MB |     false |      false |        13 |    7259394 |            7259394 |        |
| https://192.168.1.21:2379 | 4b07d8c024408053 |  3.5.12 |  7.7 MB |      true |      false |        13 |    7259394 |            7259394 |        |
+---------------------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+

```

Example view of deployments:
```bash
NAMESPACE        NAME                                           READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS    IMAGES                               SELECTOR
istio-system     deployment.apps/istio-ingressgateway           1/1     1            1           14d   istio-proxy   docker.io/istio/proxyv2:1.22.3       app=istio-ingressgateway,istio=ingressgateway
istio-system     deployment.apps/istiod                         1/1     1            1           14d   discovery     docker.io/istio/pilot:1.22.3         istio=pilot
kube-system      deployment.apps/coredns                        2/2     2            2           24d   coredns       coredns/coredns:1.9.4                k8s-app=kube-dns
metallb-system   deployment.apps/controller                     1/1     1            1           24d   controller    quay.io/metallb/controller:v0.14.7   app=metallb,component=controller
```
