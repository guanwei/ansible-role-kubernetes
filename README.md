# Ansible Role: Kubernetes

Installs Kubernetes on Debian/Ubuntu.

## Requirements

None.

## Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml`):

```
## Note that xenial repo is used for all Debian derivatives at this time.
kubernetes_apt_key: https://packages.cloud.google.com/apt/doc/apt-key.gpg
kubernetes_apt_repository: "deb http://apt.kubernetes.io/ kubernetes-xenial main"

## master or node
kubernetes_role: master

## specific kubernetes version format as 1.18.6
kubernetes_version: latest
# kubernetes_apiserver_advertise_address: "192.168.12.12"
# kubernetes_ignore_preflight_errors: "all"
kubernetes_kubeadm_init_extra_opts: ""
kubernetes_join_command_extra_opts: ""

## Note pod network cidr must be differnt with real interface network cidr.
kubernetes_pod_network:
  ## Flannel CNI.
  cni: "flannel"
  cidr: "10.244.0.0/16"
  ## Calico CNI.
  # cni: 'calico'
  # cidr: '10.244.0.0/16'

kubernetes_set_kubeconfig_for_current_user: true

kubernetes_allow_pods_on_master: true

## Flannel config files.
kubernetes_flannel_manifest_file: https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

## Calico config files
kubernetes_calico_manifest_file: https://docs.projectcalico.org/manifests/calico.yaml

## Helm
# kubernetes_helm_tiller_repo: "registry.aliyuncs.com/google_containers"
kubernetes_helm_stable_repo_url: https://kubernetes-charts.storage.googleapis.com
kubernetes_helm_migrate_releases: false

### Kubernetes apps ###

## hostpath_provisioner should be installed only for one machine k8s cluster.
kubernetes_install_hostpath_provisioner: false
kubernetes_upgrade_hostpath_provisioner: false
kubernetes_hostpath_provisioner_version: 0.2.9
kubernetes_hostpath_is_default_storage_class: true
kubernetes_hostpath_path: /mnt/hostpath

kubernetes_install_nfs_client_provisioner: false
kubernetes_upgrade_nfs_client_provisioner: false
kubernetes_nfs_client_provisioner_version: 1.2.8
kubernetes_nfs_client_is_default_storage_class: true
kubernetes_nfs_server: nfs-server
kubernetes_nfs_path: /ifs/kubernetes

kubernetes_install_nginx_ingress: false
kubernetes_upgrade_nginx_ingress: false
kubernetes_nginx_ingress_version: 1.41.2
kubernetes_nginx_ingress_external_ips:
  - "{{ kubernetes_apiserver_advertise_address | default(ansible_default_ipv4.address, true) }}"

kubernetes_install_metrics_server: false
kubernetes_upgrade_metrics_server: false
kubernetes_metrics_server_version: 2.11.1

kubernetes_install_dashboard: false
kubernetes_upgrade_dashboard: false
kubernetes_dashboard_version: 2.3.0
kubernetes_dashboard_namespace: kube-ops

kubernetes_install_prometheus_operator: false
kubernetes_upgrade_prometheus_operator: false
kubernetes_prometheus_operator_version: 9.0.1
kubernetes_prometheus_operator_namespace: kube-ops
kubernetes_prometheus_alertmanager_persistence_size: 50Gi
kubernetes_prometheus_server_persistence_size: 50Gi
kubernetes_prometheus_grafana_persistence_size: 10Gi
kubernetes_prometheus_grafana_admin_password: admin

kubernetes_install_jenkins: false
kubernetes_upgrade_jenkins: false
kubernetes_jenkins_version: 2.5.0
kubernetes_jenkins_namespace: kube-ops
kubernetes_jenkins_admin_user: admin
kubernetes_jenkins_admin_password: admin
kubernetes_jenkins_install_plugins:
  - kubernetes:1.26.4
  - workflow-aggregator:2.6
  - credentials-binding:1.23
  - git:4.3.0
  - configuration-as-code:1.42
  - job-dsl:1.77
  - active-directory:2.16
  - matrix-auth:2.6.2
  - ssh-slaves:1.31.2
  - antisamy-markup-formatter:2.1
  - ansicolor:0.7.1
  - timestamper:1.11.4
  - prometheus:2.0.7
  - maven-plugin:3.7
  - git-parameter:0.9.12
  - config-file-provider:3.6.3
  - rebuild:1.31
  - windows-slaves:1.6
  - ssh-agent:1.20
kubernetes_jenkins_casc_configs:
  system-config: |
    jenkins:
      systemMessage: Welcome to our CI\CD server.
      scmCheckoutRetryCount: 2
```

if you want use aliyun apt repo, you can set `kubernetes_apt_key` to `https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg` and set `kubernetes_apt_repository` to `deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main`.

if you want install server as k8s master, you can set `kubernetes_role` to `master`, only one server can be k8s master. Others you should set `kubernetes_role` to `node`.

if your server have more than one network card，you should set `kubernetes_apiserver_advertise_address` to the IP of the network card which you want to used.

if set `kubernetes_version` to `latest`, it will get the version from `https://storage.googleapis.com/kubernetes-release/release/stable.txt`, but if you are in china, you should be blocked, consider specific kubernetes version like `1.18.6`.

extra args for kubeadm init can be set in `kubernetes_kubeadm_init_extra_opts`. If you want pull k8s images from other docker repo set it as `--image-repository registry.cn-hangzhou.aliyuncs.com/google_containers`. 
> Becareful some new version k8s images may not exist in these repos.


`kubernetes_pod_network` can choice `flannel` or `calico` network model.

if you only have one machine to run k8s, you should set `kubernetes_allow_pods_on_master` to `true`, default is `true`.

When install tiller, you can download tiller image from other docker repo by set `kubernetes_helm_tiller_repo` to `registry.aliyuncs.com/google_containers`.

if you are blocked to access default helm stable repo, you can set `kubernetes_helm_stable_repo_url` to `https://mirror.azure.cn/kubernetes/charts/`.

if you want to migrate helm v2 releases to v3, you can set `kubernetes_helm_migrate_releases` to `true`.

this role also support install following k8s apps:
```
hostpath_provisioner
nfs_client_provisioner
nginx_ingress
metrics_server
kubernetes_dashboard
prometheus_operator
jenkins
```

you can access kubernetes_dashboard by `http://<ingress_external_ip>/kubernetes-dashboard`.

you can access prometheus server by `http://<ingress_external_ip>/prometheus`.

you can access prometheus alertmanager by `http://<ingress_external_ip>/alertmanager`.

you can access grafana by `http://<ingress_external_ip>/grafana`.

you can access jenkins by `http://<ingress_external_ip>/jenkins`.

## Dependencies

None.

## Example Playbook

requirements.yml
```
- name: kubernetes
  src: <repo_url>
  version: <branch_name>
  scm: git
```

playbook.yml
```
- hosts: servers
  roles:
    - kubernetes
```
