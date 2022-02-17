
提示信息：
```
failed to fetch PVC from API server: persistentvolumeclaims "xxx"
is forbidden ... no relationship found between node 1.0.0.1 and this object
```

初步分析：
system:node有权限，不应该查看不了

原因：
node调用API Server的API，API Server进行鉴权，查找pvc缓存失败，返回"no relationship found between node and this object"

根因：
节点禁用了ipv6，造成apiserver仅监听127.0.0.1；但api server的authorization模块的缓存是根据Pod进行更新的，而Pod则是API Server向localhost建立的连接获取到的；localhost有可能解析为::1 (节点设置了/etc/hosts双栈的解析)，因此连接建立失败。

```
controller.go:223] unable to sync kubernetes service: Post "https://[::1]:6443/api/v1/namespaces": dial tcp [::1]:6443: connect: network is unreachable
```