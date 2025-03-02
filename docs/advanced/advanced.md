---
title: "Advanced Options and Configuration"
weight: 45
aliases:
  - /k3s/latest/en/running/
  - /k3s/latest/en/configuration/
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

This section contains advanced information describing the different ways you can run and manage K3s:

- [Certificate rotation](#certificate-rotation)
- [Auto-deploying manifests](#auto-deploying-manifests)
- [Using Docker as the container runtime](#using-docker-as-the-container-runtime)
- [Using etcdctl](#using-etcdctl)
- [Configuring containerd](#configuring-containerd)
- [Running K3s with Rootless mode (Experimental)](#running-k3s-with-rootless-mode-experimental)
- [Node labels and taints](#node-labels-and-taints)
- [Starting the server with the installation script](#starting-the-server-with-the-installation-script)
- [Additional OS preparations](#additional-os-preparations)
- [Additional preparation for (Red Hat/CentOS) Enterprise Linux](#additional-preparation-for-red-hatcentos-enterprise-linux)
- [Additional preparation for Raspberry Pi OS](#additional-preparation-for-raspberry-pi-os)
- [Enabling vxlan on Ubuntu 21.10+ on Raspberry Pi](#enabling-vxlan-for-ubuntu-21.10+-on-raspberry-pi)
- [Running K3s in Docker](#running-k3s-in-docker)
- [SELinux Support](#selinux-support)
- [Enabling Lazy Pulling of eStargz (Experimental)](#enabling-lazy-pulling-of-estargz-experimental)
- [Additional Logging Sources](#additional-logging-sources)

## Certificate Rotation

By default, certificates in K3s expire in 12 months.

If the certificates are expired or have fewer than 90 days remaining before they expire, the certificates are rotated when K3s is restarted.

## Auto-Deploying Manifests

Any file found in `/var/lib/rancher/k3s/server/manifests` will automatically be deployed to Kubernetes in a manner similar to `kubectl apply`, both on startup and when the file is changed on disk. Deleting files out of this directory will not delete the corresponding resources from the cluster.

For information about deploying Helm charts, refer to the section about [Helm.](helm/helm.md)

## Using Docker as the Container Runtime

K3s includes and defaults to [containerd,](https://containerd.io/) an industry-standard container runtime.

To use Docker instead of containerd,

1. Install Docker on the K3s node. One of Rancher's [Docker installation scripts](https://github.com/rancher/install-docker) can be used to install Docker:

    ```bash
    curl https://releases.rancher.com/install-docker/20.10.sh | sh
    ```

2. Install K3s using the `--docker` option:

    ```bash
    curl -sfL https://get.k3s.io | sh -s - --docker
    ```

3. Confirm that the cluster is available:

    ```bash
    $ sudo k3s kubectl get pods --all-namespaces
    NAMESPACE     NAME                                     READY   STATUS      RESTARTS   AGE
    kube-system   local-path-provisioner-6d59f47c7-lncxn   1/1     Running     0          51s
    kube-system   metrics-server-7566d596c8-9tnck          1/1     Running     0          51s
    kube-system   helm-install-traefik-mbkn9               0/1     Completed   1          51s
    kube-system   coredns-8655855d6-rtbnb                  1/1     Running     0          51s
    kube-system   svclb-traefik-jbmvl                      2/2     Running     0          43s
    kube-system   traefik-758cd5fc85-2wz97                 1/1     Running     0          43s
    ```

4. Confirm that the Docker containers are running:

    ```bash
    $ sudo docker ps
    CONTAINER ID        IMAGE                     COMMAND                  CREATED              STATUS              PORTS               NAMES
    3e4d34729602        897ce3c5fc8f              "entry"                  About a minute ago   Up About a minute                       k8s_lb-port-443_svclb-traefik-jbmvl_kube-system_d46f10c6-073f-4c7e-8d7a-8e7ac18f9cb0_0
    bffdc9d7a65f        rancher/klipper-lb        "entry"                  About a minute ago   Up About a minute                       k8s_lb-port-80_svclb-traefik-jbmvl_kube-system_d46f10c6-073f-4c7e-8d7a-8e7ac18f9cb0_0
    436b85c5e38d        rancher/library-traefik   "/traefik --configfi…"   About a minute ago   Up About a minute                       k8s_traefik_traefik-758cd5fc85-2wz97_kube-system_07abe831-ffd6-4206-bfa1-7c9ca4fb39e7_0
    de8fded06188        rancher/pause:3.1         "/pause"                 About a minute ago   Up About a minute                       k8s_POD_svclb-traefik-jbmvl_kube-system_d46f10c6-073f-4c7e-8d7a-8e7ac18f9cb0_0
    7c6a30aeeb2f        rancher/pause:3.1         "/pause"                 About a minute ago   Up About a minute                       k8s_POD_traefik-758cd5fc85-2wz97_kube-system_07abe831-ffd6-4206-bfa1-7c9ca4fb39e7_0
    ae6c58cab4a7        9d12f9848b99              "local-path-provisio…"   About a minute ago   Up About a minute                       k8s_local-path-provisioner_local-path-provisioner-6d59f47c7-lncxn_kube-system_2dbd22bf-6ad9-4bea-a73d-620c90a6c1c1_0
    be1450e1a11e        9dd718864ce6              "/metrics-server"        About a minute ago   Up About a minute                       k8s_metrics-server_metrics-server-7566d596c8-9tnck_kube-system_031e74b5-e9ef-47ef-a88d-fbf3f726cbc6_0
    4454d14e4d3f        c4d3d16fe508              "/coredns -conf /etc…"   About a minute ago   Up About a minute                       k8s_coredns_coredns-8655855d6-rtbnb_kube-system_d05725df-4fb1-410a-8e82-2b1c8278a6a1_0
    c3675b87f96c        rancher/pause:3.1         "/pause"                 About a minute ago   Up About a minute                       k8s_POD_coredns-8655855d6-rtbnb_kube-system_d05725df-4fb1-410a-8e82-2b1c8278a6a1_0
    4b1fddbe6ca6        rancher/pause:3.1         "/pause"                 About a minute ago   Up About a minute                       k8s_POD_local-path-provisioner-6d59f47c7-lncxn_kube-system_2dbd22bf-6ad9-4bea-a73d-620c90a6c1c1_0
    64d3517d4a95        rancher/pause:3.1         "/pause"
    ```

## Using etcdctl

etcdctl provides a CLI for etcd.

If you would like to use etcdctl after installing K3s with embedded etcd, install etcdctl using the [official documentation.](https://etcd.io/docs/latest/install/) 

```bash
$ VERSION="v3.5.0"
$ curl -L https://github.com/etcd-io/etcd/releases/download/${VERSION}/etcd-${VERSION}-linux-amd64.tar.gz --output etcdctl-linux-amd64.tar.gz
$ sudo tar -zxvf etcdctl-linux-amd64.tar.gz --strip-components=1 -C /usr/local/bin etcd-${VERSION}-linux-amd64/etcdctl
```

Then start using etcdctl commands with the appropriate K3s flags:

```bash
sudo etcdctl --cacert=/var/lib/rancher/k3s/server/tls/etcd/server-ca.crt --cert=/var/lib/rancher/k3s/server/tls/etcd/client.crt --key=/var/lib/rancher/k3s/server/tls/etcd/client.key version
```

## Configuring containerd

K3s will generate config.toml for containerd in `/var/lib/rancher/k3s/agent/etc/containerd/config.toml`.

For advanced customization for this file you can create another file called `config.toml.tmpl` in the same directory and it will be used instead.

The `config.toml.tmpl` will be treated as a Go template file, and the `config.Node` structure is being passed to the template. See [this folder](https://github.com/k3s-io/k3s/blob/master/pkg/agent/templates) for Linux and Windows examples on how to use the structure to customize the configuration file.


## Running K3s with Rootless mode (Experimental)

> **Warning:** This feature is experimental.

Rootless mode allows running the entire k3s an unprivileged user, so as to protect the real root on the host from potential container-breakout attacks.

See also https://rootlesscontaine.rs/ to learn about Rootless mode.

### Known Issues with Rootless mode

* **Ports**

    When running rootless a new network namespace is created.  This means that K3s instance is running with networking fairly detached from the host.  The only way to access services run in K3s from the host is to set up port forwards to the K3s network namespace. We have a controller that will automatically bind 6443 and service port below 1024 to the host with an offset of 10000. 

    That means service port 80 will become 10080 on the host, but 8080 will become 8080 without any offset.

    Currently, only `LoadBalancer` services are automatically bound.

* **Cgroups**

    Cgroup v1 is not supported. v2 is supported.

* **Multi-node cluster**

    Multi-cluster installation is untested and undocumented.

### Running Servers and Agents with Rootless
* Enable cgroup v2 delegation, see https://rootlesscontaine.rs/getting-started/common/cgroup2/ .
  This step is optional, but highly recommended for enabling CPU and memory resource limtitation.

* Download `k3s-rootless.service` from [`https://github.com/k3s-io/k3s/blob/<VERSION>/k3s-rootless.service`](https://github.com/k3s-io/k3s/blob/master/k3s-rootless.service).
  Make sure to use the same version of `k3s-rootless.service` and `k3s`.

* Install `k3s-rootless.service` to `~/.config/systemd/user/k3s-rootless.service`.
  Installing this file as a system-wide service (`/etc/systemd/...`) is not supported.
  Depending on the path of `k3s` binary, you might need to modify the `ExecStart=/usr/local/bin/k3s ...` line of the file.

* Run `systemctl --user daemon-reload`

* Run `systemctl --user enable --now k3s-rootless`

* Run `KUBECONFIG=~/.kube/k3s.yaml kubectl get pods -A`, and make sure the pods are running.

> **Note:** Don't try to run `k3s server --rootless` on a terminal, as it doesn't enable cgroup v2 delegation.
> If you really need to try it on a terminal, prepend `systemd-run --user -p Delegate=yes --tty` to create a systemd scope.
>
> i.e., `systemd-run --user -p Delegate=yes --tty k3s server --rootless`

### Troubleshooting

* Run `systemctl --user status k3s-rootless` to check the daemon status
* Run `journalctl --user -f -u k3s-rootless` to see the daemon log
* See also https://rootlesscontaine.rs/

## Node Labels and Taints

K3s agents can be configured with the options `--node-label` and `--node-taint` which adds a label and taint to the kubelet. The two options only add labels and/or taints [at registration time,](reference/agent-config.md#node-labels-and-taints-for-agents) so they can only be added once and not changed after that again by running K3s commands.

If you want to change node labels and taints after node registration you should use `kubectl`. Refer to the official Kubernetes documentation for details on how to add [taints](https://kubernetes.io/docs/concepts/configuration/taint-and-toleration/) and [node labels.](https://kubernetes.io/docs/tasks/configure-pod-container/assign-pods-nodes/#add-a-label-to-a-node)

## Starting the Server with the Installation Script

The installation script will auto-detect if your OS is using systemd or openrc and start the service.
When running with openrc, logs will be created at `/var/log/k3s.log`. 

When running with systemd, logs will be created in `/var/log/syslog` and viewed using `journalctl -u k3s`.

An example of installing and auto-starting with the install script:

```bash
curl -sfL https://get.k3s.io | sh -
```

When running the server manually you should get an output similar to the following:

```bash
$ k3s server
INFO[2019-01-22T15:16:19.908493986-07:00] Starting k3s dev                             
INFO[2019-01-22T15:16:19.908934479-07:00] Running kube-apiserver --allow-privileged=true --authorization-mode Node,RBAC --service-account-signing-key-file /var/lib/rancher/k3s/server/tls/service.key --service-cluster-ip-range 10.43.0.0/16 --advertise-port 6445 --advertise-address 127.0.0.1 --insecure-port 0 --secure-port 6444 --bind-address 127.0.0.1 --tls-cert-file /var/lib/rancher/k3s/server/tls/localhost.crt --tls-private-key-file /var/lib/rancher/k3s/server/tls/localhost.key --service-account-key-file /var/lib/rancher/k3s/server/tls/service.key --service-account-issuer k3s --api-audiences unknown --basic-auth-file /var/lib/rancher/k3s/server/cred/passwd --kubelet-client-certificate /var/lib/rancher/k3s/server/tls/token-node.crt --kubelet-client-key /var/lib/rancher/k3s/server/tls/token-node.key 
Flag --insecure-port has been deprecated, This flag will be removed in a future version.
INFO[2019-01-22T15:16:20.196766005-07:00] Running kube-scheduler --kubeconfig /var/lib/rancher/k3s/server/cred/kubeconfig-system.yaml --port 0 --secure-port 0 --leader-elect=false
INFO[2019-01-22T15:16:20.196880841-07:00] Running kube-controller-manager --kubeconfig /var/lib/rancher/k3s/server/cred/kubeconfig-system.yaml --service-account-private-key-file /var/lib/rancher/k3s/server/tls/service.key --allocate-node-cidrs --cluster-cidr 10.42.0.0/16 --root-ca-file /var/lib/rancher/k3s/server/tls/token-ca.crt --port 0 --secure-port 0 --leader-elect=false 
Flag --port has been deprecated, see --secure-port instead.
INFO[2019-01-22T15:16:20.273441984-07:00] Listening on :6443                           
INFO[2019-01-22T15:16:20.278383446-07:00] Writing manifest: /var/lib/rancher/k3s/server/manifests/coredns.yaml 
INFO[2019-01-22T15:16:20.474454524-07:00] Node token is available at /var/lib/rancher/k3s/server/node-token 
INFO[2019-01-22T15:16:20.474471391-07:00] To join node to cluster: k3s agent -s https://10.20.0.3:6443 -t ${NODE_TOKEN} 
INFO[2019-01-22T15:16:20.541027133-07:00] Wrote kubeconfig /etc/rancher/k3s/k3s.yaml
INFO[2019-01-22T15:16:20.541049100-07:00] Run: k3s kubectl                             
```

The output will likely be much longer as the agent will create a lot of logs. By default, the server
will register itself as a node (run the agent).

## Additional OS Preparations

### Additional preparation for Debian "buster" based distributions

Several popular Linux distributions based on Debian "buster" ship a version of iptables between v1.8.0-v1.8.4. These versions contain a bug which causes the accumulation of duplicate rules, which negatively affects the performance and stability of the node. See [Issue #3117](https://github.com/k3s-io/k3s/issues/3117) for more background. 

Switching from nftables mode to legacy iptables mode will bypass this issue.

```bash
sudo iptables -F
sudo update-alternatives --set iptables /usr/sbin/iptables-legacy
sudo update-alternatives --set ip6tables /usr/sbin/ip6tables-legacy
sudo reboot
```

Alternatively, K3s ships which a working version of ipTables (v1.8.6) which functions properly. You can replace iptables on your system:

```bash
sudo apt remove iptables nftables -y
sudo reboot
export PATH="/var/lib/rancher/k3s/data/current/bin/:/var/lib/rancher/k3s/data/current/bin/aux:$PATH"
```

K3s will now use its packaged version of iptables.

```bash
$ which iptables
/var/lib/rancher/k3s/data/current/bin/aux/iptables
```

### Additional preparation for (Red Hat/CentOS) Enterprise Linux

It is recommended to turn off firewalld:
```bash
systemctl disable firewalld --now
```

If enabled, it is required to disable nm-cloud-setup and reboot the node:
```bash
systemctl disable nm-cloud-setup.service nm-cloud-setup.timer
reboot
```

### Additional preparation for Raspberry Pi OS

Raspberry Pi OS is Debian based, and may suffer from an old iptables version. See [workarounds](#additional-preparation-for-debian-buster-based-distributions).

Standard Raspberry Pi OS installations do not start with `cgroups` enabled. **K3S** needs `cgroups` to start the systemd service. `cgroups`can be enabled by appending `cgroup_memory=1 cgroup_enable=memory` to `/boot/cmdline.txt`.

Example cmdline.txt:
```
console=serial0,115200 console=tty1 root=PARTUUID=58b06195-02 rootfstype=ext4 elevator=deadline fsck.repair=yes rootwait cgroup_memory=1 cgroup_enable=memory
```

## Enabling vxlan for Ubuntu 21.10+ on Raspberry Pi

Starting with Ubuntu 21.10, vxlan support on Raspberry Pi has been moved into a separate kernel module. 
```bash
sudo apt install linux-modules-extra-raspi
```

## Running K3s in Docker

There are several ways to run K3s in Docker:

<Tabs>
<TabItem value='K3d' default>

[k3d](https://github.com/k3s-io/k3d) is a utility designed to easily run K3s in Docker.

It can be installed via the the [brew](https://brew.sh/) utility on MacOS:

```bash
brew install k3d
```

</TabItem>
<TabItem value="Docker Compose">

A `docker-compose.yml` in the K3s repo serves as an [example](https://github.com/k3s-io/k3s/blob/master/docker-compose.yml) of how to run K3s from Docker. To run `docker-compose` in this repo, run:

```bash
docker-compose up --scale agent=3
# kubeconfig is written to current dir

kubectl --kubeconfig kubeconfig.yaml get node

NAME           STATUS   ROLES    AGE   VERSION
497278a2d6a2   Ready    <none>   11s   v1.13.2-k3s2
d54c8b17c055   Ready    <none>   11s   v1.13.2-k3s2
db7a5a5a5bdd   Ready    <none>   12s   v1.13.2-k3s2
```

To only run the agent in Docker, use `docker-compose up agent`.

</TabItem>
<TabItem value="Docker">

To use Docker, `rancher/k3s` images are also available to run the K3s server and agent. 
Using the `docker run` command:

```bash
sudo docker run \
  -d --tmpfs /run \
  --tmpfs /var/run \
  -e K3S_URL=${SERVER_URL} \
  -e K3S_TOKEN=${NODE_TOKEN} \
  --privileged rancher/k3s:vX.Y.Z
```
</TabItem>
</Tabs>

## SELinux Support

:::info Version Gate

Available as of v1.19.4+k3s1

:::

If you are installing K3s on a system where SELinux is enabled by default (such as CentOS), you must ensure the proper SELinux policies have been installed. 

<Tabs>
<TabItem value="Automatic Installation" default>

The [install script](installation/configuration#options-for-installation-with-script) will automatically install the SELinux RPM from the Rancher RPM repository if on a compatible system if not performing an air-gapped install. Automatic installation can be skipped by setting `INSTALL_K3S_SKIP_SELINUX_RPM=true`.

</TabItem>

<TabItem value="Manual Installation" default>

The necessary policies can be installed with the following commands:
```bash
yum install -y container-selinux selinux-policy-base
yum install -y https://rpm.rancher.io/k3s/latest/common/centos/7/noarch/k3s-selinux-0.2-1.el7_8.noarch.rpm
```

To force the install script to log a warning rather than fail, you can set the following environment variable: `INSTALL_K3S_SELINUX_WARN=true`.
</TabItem>
</Tabs>

### Enabling SELinux Enforcement

To leverage SELinux, specify the `--selinux` flag when starting K3s servers and agents.
  
This option can also be specified in the K3s [configuration file](#).

```
selinux: true
```

Using a custom `--data-dir` under SELinux is not supported. To customize it, you would most likely need to write your own custom policy. For guidance, you could refer to the [containers/container-selinux](https://github.com/containers/container-selinux) repository, which contains the SELinux policy files for Container Runtimes, and the [rancher/k3s-selinux](https://github.com/rancher/k3s-selinux) repository, which contains the SELinux policy for K3s.

## Enabling Lazy Pulling of eStargz (Experimental)

### What's lazy pulling and eStargz?

Pulling images is known as one of the time-consuming steps in the container lifecycle.
According to [Harter, et al.](https://www.usenix.org/conference/fast16/technical-sessions/presentation/harter),

> pulling packages accounts for 76% of container start time, but only 6.4% of that data is read

To address this issue, k3s experimentally supports *lazy pulling* of image contents.
This allows k3s to start a container before the entire image has been pulled.
Instead, the necessary chunks of contents (e.g. individual files) are fetched on-demand. 
Especially for large images, this technique can shorten the container startup latency.

To enable lazy pulling, the target image needs to be formatted as [*eStargz*](https://github.com/containerd/stargz-snapshotter/blob/main/docs/stargz-estargz.md).
This is an OCI-alternative but 100% OCI-compatible image format for lazy pulling.
Because of the compatibility, eStargz can be pushed to standard container registries (e.g. ghcr.io) as well as this is *still runnable* even on eStargz-agnostic runtimes.

eStargz is developed based on the [stargz format proposed by Google CRFS project](https://github.com/google/crfs) but comes with practical features including content verification and performance optimization.
For more details about lazy pulling and eStargz, please refer to [Stargz Snapshotter project repository](https://github.com/containerd/stargz-snapshotter).

### Configure k3s for lazy pulling of eStargz

As shown in the following, `--snapshotter=stargz` option is needed for k3s server and agent.

```bash
k3s server --snapshotter=stargz
```

With this configuration, you can perform lazy pulling for eStargz-formatted images.
The following Pod manifest uses eStargz-formatted `node:13.13.0` image (`ghcr.io/stargz-containers/node:13.13.0-esgz`).
k3s performs lazy pulling for this image.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nodejs
spec:
  containers:
  - name: nodejs-estargz
    image: ghcr.io/stargz-containers/node:13.13.0-esgz
    command: ["node"]
    args:
    - -e
    - var http = require('http');
      http.createServer(function(req, res) {
        res.writeHead(200);
        res.end('Hello World!\n');
      }).listen(80);
    ports:
    - containerPort: 80
```

## Additional Logging Sources

[Rancher logging](https://rancher.com/docs/rancher/v2.6/en/logging/helm-chart-options/) for K3s can be installed without using Rancher. The following instructions should be executed to do so:

```bash
helm repo add rancher-charts https://charts.rancher.io
helm repo update
helm install --create-namespace -n cattle-logging-system rancher-logging-crd rancher-charts/rancher-logging-crd
helm install --create-namespace -n cattle-logging-system rancher-logging --set additionalLoggingSources.k3s.enabled=true rancher-charts/rancher-logging
```
