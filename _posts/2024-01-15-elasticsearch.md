---
layout: post
title: elasticsearch
subtitle: for kubernetes
cover-img: /assets/img/path.jpg
thumbnail-img: /assets/img/thumb.png
share-img: /assets/img/path.jpg
tags: [kubernetes, config, database, elasticsearch]
---
### elasticsearch
#### elasticsearch.yml
1. 要使用fleet就要打开安全，集群模式下打开安全后需要ssl证书。设置为单节点后启动不需要配置ssl 证书(xpack.security.transport.ssl.enabled)。
  ```yaml
  #单节点模式
  discovery.type: single-node
  # 启用安全
  xpack.security.enabled: true
  ```
#### jvmoptions

1. 设置内存大小，最大和最小必须都设置成一样才生效。

```yaml
    -Xms16g

    -Xmx16g
```

### kibana

1. 需要使用elasticsearch-reset-password -i-u kibana_system来重置账号密码。

### fleet

1. fleet需要包注册仓库，需要启动仓库distribution
