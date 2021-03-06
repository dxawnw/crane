# 注意事项

当前 Kubernetes 部署过程中遇到的问题可以通过如下描述进行解决：

### 无法通过 nodePort 访问服务

如发生此类问题时修改 Kube-Proxy 中 node_port_address 地址即可，或通过 crane 的 kube_proxy_node_port_addresses 修改。

### 不要对 pause 进行升级

如果集群是正在使用的不要对 pause 进行更新，它会重启所有的 Pod。

### 使用 Calico IPIP 更换 BGP

使用 Calico 时默认是 IPIP 协议，如果需要修改 BGP 只需要对 Crane 的 calico_type 改为 Off 即可生效。

> 但如果是正在使用的集群，启动后删除 tunl0 网卡后才会生效，这里面存在网络冲突。


### CNI/ClusterIP 变更

CNI 变更修改 Calico 中的 CALICO_IPV4POOL_CIDR 后重启所有 Pod 生效，或者不重启因 iptables 等规则已经生效且等待下一次更新即可。

ClusterIP 地址变更需要对所有的 Kube-apiServer 的 service-cluster-ip-range 参数进行变更，然后重启 apiServer 即可生效。

ClustarIP 变更后需要对 DNS 进行地址变更，首先修改 DNS Service 的 ClusterIP, 然后变更所有 kubelet 配置，执行如下：

```
$ sed -i 's/10.96.0.10/10.9.0.10/g' /etc/systemd/system/kubelet.service.d/10-kubelet.conf
$ sed -i 's/10.96.0.10/10.9.0.10/g' /var/lib/kubelet/config.yaml
$ systemctl daemon-reload
$ systemctl restart kubelet
```

执行完毕后整个变更成功。


### Jenkins Build 403 Error

部署 Jenkins 后通过 Ingress 暴露服务访问并构建时，存在 403 错误，请使用 Calico 的 BGP 模式解决。