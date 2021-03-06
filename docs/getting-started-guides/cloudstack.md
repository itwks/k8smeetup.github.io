---
assignees:
- thockin
title: Cloudstack
---


[CloudStak](https://cloudstack.apache.org/) 是一个基于硬件虚拟技术的共有和私有云(传统的 Iaas). 在 CloundStack 上部署 Kubernetes 取决于云所运用的依赖关系以及使用的镜像。CloudStack 同样提供了 vagrant 插件，Vagrant 能够用于使用现有的 shell 或者新的 Salt based 方法部署 Kubernetes。


[CoreOS](http://coreos.com) CloudStack 模版会每天晚上重新构建， 每一位 CloudStack 操作者在设计 Kubernetes 部署框架前都需要在自己的云上[注册](http://docs.cloudstack.apache.org/projects/cloudstack-administration/en/latest/templates.html)模版


这份指导文档里使用的是[Ansible playbook](https://github.com/apachecloudstack/k8s).
在这里使用完全自动化的，单一的脚本部署 Kubernetes

这里的 [Ansible](http://ansibleworks.com) playbook 在 CloudStack 云上使用 CoreOS 镜像部署 Kubernetes。 脚本会： 创建一个 ssh 钥匙，创建一个安全组并且配置相应的访问规则，最后启动一个通过 cloud-init 配置的 coreOS 实例


* TOC
{:toc}


## 准备工作

    $ sudo apt-get install -y python-pip libssl-dev
    $ sudo pip install cs
    $ sudo pip install sshpubkeys
    $ sudo apt-get install software-properties-common
    $ sudo apt-add-repository ppa:ansible/ansible
    $ sudo apt-get update
    $ sudo apt-get install ansible


在 CloudStack 服务器上你还需要安装 libselinux-python :

    yum install libselinux-python

[_cs_](https://github.com/exoscale/cs) 这是一个python版本的 CloudStack API.
如果想随时查看 CloudStack 端点使用的 API 和 HTTP ，你可以把它们定义成如下的环境变量：`CLOUDSTACK_ENDPOINT`, `CLOUDSTACK_KEY`, `CLOUDSTACK_SECRET` 和 `CLOUDSTACK_METHOD`.
或者创建一个 `~/.cloudstack.ini` 文件：

    [cloudstack]
    endpoint = <your cloudstack api endpoint>
    key = <your api access key>
    secret = <your api secret key>
    method = post


我们需要使用 http 的 POST 方法传递 _large_ userdata 给 coreOS 的实例。
## 克隆 playbook
    $ git clone https://github.com/apachecloudstack/k8s
    $ cd kubernetes-cloudstack

###  创建一个 Kubernetes 集群
只需要简单的运行 playbook就可以。
   $ ansible-playbook k8s.yml

某些变量的值可以通过编辑 `k8s.yml` 文件来修改。

    vars:
      ssh_key: k8s
      k8s_num_nodes: 2
      k8s_security_group_name: k8s
      k8s_node_prefix: k8s2
      k8s_template: <templatename>
      k8s_instance_type: <serviceofferingname>


下面我们将启动一个 Kubernetes master 节点以及一系列计算节点（默认是2个）。
`instance_type` 和 `template` 的值用于指定 CloudStack 云上特定的模版和实例的类型（比如说 ： 提供服务）

如果你想修改某些变量值，请参考在 `roles/k8s` 里的任务和模版。

一旦 playbook 运行结束，它将返回类似如下的 Kubernetes master 的 IP:
    TASK: [k8s | debug msg='k8s master IP is {% raw %}{{ k8s_master.default_ip }}{% endraw %}'] ********

使用以上 _core_ user 的建，我们可以通过 SSH 的方式的访问 Kubernetes master，比如说可以列举出来当前集群的机器：

    $ ssh -i ~/.ssh/id_rsa_k8s core@<master IP>
    $ fleetctl list-machines
    MACHINE        IP               METADATA
    a017c422...    <node #1 IP>   role=node
    ad13bf84...    <master IP>       role=master
    e9af8293...    <node #2 IP>   role=node

## 支持

aaS Provider        | Config. Mgmt | OS     | Networking  | Docs                                              | Conforms | Support Level
-------------------- | ------------ | ------ | ----------  | ---------------------------------------------     | ---------| ----------------------------
CloudStack           | Ansible      | CoreOS | flannel     | [docs](/docs/getting-started-guides/cloudstack)                             |          | Community ([@Guiques](https://github.com/ltupin/))

不同解决方案的支持信息， 请参见表格[Table of solutions](/docs/getting-started-guides/#table-of-solutions)