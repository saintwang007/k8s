# k8s modify for etcdv3 basic auth

## etcdv3 basic auth
etcd有v2和v3版本，网上搜索的配置基本上是v2版本的，v2和v3设置互不影响。（意思是如果你设置v2版本的basic auth，对v3版本的权限控制并不生效） 

1.添加root管理员账号<br>

  ```
  ETCDCTL_API=3 etcdctl user add root
  ```

2.开启认证<br>

  ```
  ETCDCTL_API=3 etcdctl auth enable
  ```

3.创建读写角色<br>
  
  ```
  ETCDCTL_API=3 etcdctl  --user root  role grant-permission readwrite --prefix=true readwrite ''
  ETCDCTL_API=3 etcdctl  --user root  role grant-permission read --prefix=true read ''
  ```

4.创建读写用户，并授予读写权限<br>

  ```
  ETCDCTL_API=3 etcdctl  --user root:password  user add readwriter
  ETCDCTL_API=3 etcdctl  --user root:password  user add reader
  ETCDCTL_API=3 etcdctl  --user root:password  user grant-role readwriter readwrite
  ETCDCTL_API=3 etcdctl  --user root:password  user grant-role reader read
  ```
  
5.关闭认证<br>
  
  ```
  ETCDCTL_API=3 etcdctl  --user root:password auth disable
  ```
  
参考连接：https://github.com/etcd-io/etcd/blob/master/Documentation/op-guide/authentication.md

## k8s code modify
本修改仅支持etcdv3的basic auth。 

1.下载kubenetes1.13.0源码并替换掉源码中下面几个文件<br>

```
./pkg/version/base.go
./staging/src/k8s.io/apiserver/pkg/server/options/etcd.go
./staging/src/k8s.io/apiserver/pkg/storage/storagebackend/factory/etcd3.go
./staging/src/k8s.io/apiserver/pkg/storage/storagebackend/config.go
```

2.然后编译kube-apiserver二进制,就可使用。(本文已编译好一个，放在/bin/目录下)<br>

3.执行`./kube-apiserver --help`,会看到增加这么一行:<br>

`--etcd-baseauth string                     BaseAuth file used to secure etcd communication.`

4.开启etcd basic auth:<br>

`./kube-apiserver --etcd-baseauth=/etc/kubernetes/security/etcd_baseauth.csv`

5.etcd_baseauth.csv文件内容格式如下：<br>

`username1,password1`

备注：时间有限，仓促整理，如果说明不到位敬请谅解！
