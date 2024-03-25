## 引言

本文是《深入剖析 K8s》的学习笔记，相关图片和案例可从[https://github.com/WeiXiao-Hyy/k8s_example](https://github.com/WeiXiao-Hyy/k8s_example)中获取，欢迎 :star:!

## K8s 的网络隔离: NetWorkPolicy

K8s 如何考虑容器之间网络的“隔离” -> NetWorkPolicy

以下是一个 NetWorkPolicy 的定义。

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
  namespace: default
spec:
  # 指定套用 network policy 的 pod
  # 若沒指定 podSelector，就表示將 network policy 套用到 namespace 中的所有 pod
  podSelector:
    matchLabels:
      role: db
  # 設定 network policy 包含的 policy type 有那些
  policyTypes:
    - Ingress
    - Egress
  # ingress 用來設定從外面進來的流量的白名單
  ingress:
    - from:
        # 指定 IP range
        - ipBlock:
            cidr: 172.17.0.0/16
            # 例外設定
            except:
              - 172.17.1.0/24
        # 帶有特定 label 的 namespace 中所有的 pod
        - namespaceSelector:
            matchLabels:
              project: myproject
        # 同一個 namespace 中帶有特定 label 的 pod
        - podSelector:
            matchLabels:
              role: frontend
      ports:
        - protocol: TCP
          port: 6379
  # egress 用來設定 pod 對外連線的白名單
  egress:
    - to:
        - ipBlock:
            cidr: 10.0.0.0/24
      ports:
        - protocol: TCP
          port: 5978
```

> ingress, exgress 的行为

- `ingress` 搭配 from, 负责管理从外部进来的流量
- `exgress` 搭配 to, 负责管理从内部出去的流量

> 设定 Default Policy

- 若要选择所有的 pod，则将 `podSelector` 设定为 {}
- Network Policy 的设定，若是设定 `policyType`，就表示要全部拒绝该种连接(`ingress/egress`)的流量
- 若要打开上面的限制，則要在 `spec.ingress` or `spec.egress` 中设定

凡是支持 NetWorkPolicy 的 CNI 网络插件，都维护着一个 NetWorkPolicy Controller，通过控制循环的方式对 NetWorkPolicy 对象的 CRUD 做出响应，然后在宿主机上完成 iptables 规则的配置工作。

> 案例

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: deny-egress-test
  labels:
    app: "deny-egress-test"
  namespace: "new-ns"
spec:
  containers:
    - name: nettools
      image: travelping/nettools
      command: ["sh", "-c", "sleep 3600"]
```

正常的 pod 可以访问外界。

![pod-ping](../imgs/pod-ping.png)

添加如下 NetWorkPolicy 之后，ping 失败。

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-egress-test
  namespace: "new-ns"
spec:
  podSelector:
    matchLabels:
      app: "deny-egress-test"
  policyTypes:
    - Egress
```

![pod-ping-failed](../imgs/pod-ping-failed.png)

> iptables

iptables 相关配置可以参考[https://zh.wikipedia.org/wiki/Iptables](https://zh.wikipedia.org/wiki/Iptables)

## 参考资料

- [https://godleon.github.io/blog/Kubernetes/k8s-Network-Policy-Overview/](https://godleon.github.io/blog/Kubernetes/k8s-Network-Policy-Overview/)
- [https://zh.wikipedia.org/wiki/Iptables](https://zh.wikipedia.org/wiki/Iptables)
