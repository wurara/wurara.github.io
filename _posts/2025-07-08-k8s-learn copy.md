---
layout: post
title: k8s learn
subtitle: 学习k8s
cover-img: /assets/img/path.jpg
thumbnail-img: /assets/img/thumb.png
share-img: /assets/img/path.jpg
tags: [learn]
---



5. 





5. 部署
   1. statefulSet
      1. 有序创建，倒序回收
      2. 数据持久化pod级别：每个pod重启后拿到的pv是一致的
      3. 稳定的网络方式：
         1. podName.headlessSvcName.nsName.svc.domainName
         2. statefulSetName-num.headless
             SvcName.nsName.svc.domainName
         3. web-0.nginx.  .svc.cluster.local



6. serivce：iptables或者ipvs

   1. clusterip
      1. internalTrafficPolicy： 请求只路由当前节点的服务的port
      2. sessionAffinity：长连接
      3. publishNotReadyAddresses: 访问未就绪pod
   2. nodeport
      1. externalTrafficPolicy：： 请求只路由当前节点的服务的port
   3. LoadBalancer
      1. 由云服务厂商提供
   4. ExternalName
   5. Endpoint

7. 存储
   1. configMap
      1. 注入式的
      2. --from-file： 从文件读取
      3. --dry-run -o yaml >2.yaml：将运行结果写到文件中
      4. --from-literal
      5. 挂载为环境变量或者挂在为文件
      6. 文件是软连接的形式，便于做热更新。环境变量不会热更新。
      7. immutable: 不可改变。不可逆。

   2. Secret
      1. 只有使用的pod会获取
      2. 只存在于内存中
      3. etcd以加密形式存储
      4. 值必须经过base64编码
      5. 存储在卷中的可以被热更新。环境变量不会被热更新
      6. immutable: 不可改变。不可逆。
      7. 形式
         1. Opaque
            
            ```yaml
            ---
            #此类卷挂载到服务器上时不会热更新
            volumes:
            	name: volumesl2 secret:
            		secretName: mysecret
            		defaultMode: 256
            ---
            volumes:
            	name: volumesl2 
            	secret:
            		secretName: mysecret
            		items:
                    	- key: username
            			 path: my-group/my-username
            ```
            

   3. DownwardApi

      1. 热更新

      2. 可以暴露apiserver接口实现访问部署

      3.  设置环境变量

         ```yaml
         env:
         - name: PDO_NAME
           valueFrom:
           	fieldRef:
           		fieldPath: metadata.name
         ```

         

      4. 挂载卷

         ```yaml
         	vloumeMounts:
         		- name: downward-api-volume
         	  	  mountPath: /etc/podinfo
         volumes:
         - name: downward-api-volume
           downwardAPI:
           	items:
           	- path: "annotations"
           	  fieldRef:
           	  	FieldPath: metadata.annotations
         ```

         

   4. Volume

      1. emptyDir

         1. 生命周期和pod一致

         2. 真实路径是/var/lib/kubelet/pods/{podid}/volumes/kubernetes.io~empty-dir/

         3. 内存形式

            ```yaml
            volumes:
            - name: mem-voluime
              emptyDir:
              	medium: memory
              	sizeLimit: 50Mi
            ```

      2. hostrPath

         1. 可能有权限问题
         2. 路径类型检查
            1. 空
            2. DirectoryorCreate
            3. Directory
            4. FileOrCreate
            5. File
            6. Socket
            7. CharDevice
            8. BlockDevice

   5. PersisterntVolume

      1. pv容量不小于pvc
      2. 读写类型必须一致。单节点读写、多节点读写、多节点只读
      3. 存储类：pv和pvc必须一致
      4. 回收策略：
         1. 保留：需要手动删除
         2. 回收：删除内部基础数据（nfs和hostpath支持）
         3. 删除：删除相关pv（云厂商支持）
      5. 状态
         1. 可用
         2. 已绑定
         3. 已释放:这个状态的pv不可用。需要删除pv或者删除pv的绑定信息
         4. 失败
      6. 保护策略
         1. pvc需要手动删除
         2. pvc存在时不允许删除pv
         3. 删除pvc时只有使用方被删除了才会释放pvc

   6. storageClass
      1. nfs-client-provisioner

8. 调度器

   1. 要求
      1. 公平
      2. 资源高效利用
      3. 效率
      4. 灵活
   
   2. 调度过程
      1. 预选
         1. PodFitsResources:剩余资源大于pod请求资源
         2. PodFitsHost：是否指定了node
         3. PodFitsHostPorts：port是否和pod冲突
         4. PodSelectorMatches：label是否匹配
         5. NoDiskConflict：mount的volume是否冲突
      2. 优选
         1. LeastRequestedPriority:CPU和Memory使用率越低
         2. BalanceResourceAllocation：CPU和Memory使用率越接近
         3. ImageLocalityuPriority：已经有镜像的总占用大小，越大越高
   
   3. 亲和性
   
      1. node亲和性，pod亲和性，pod反亲和性
   
      2. 软亲和：preferredDuringSchedulingIgnoredDuringExecution
   
      3. 硬亲和：requiredDuringSchedulingIgnoredDuringExecution
   
      4. 反软亲和
   
         preferredDuringSchedulingIgnoredDuringExecution
   
      5. 反硬亲和
   
         requiredDuringSchedulingIgnoredDuringExecution
   
      6. topologyKey：节点信息
   
         1. kubernetes.io/hostname‌（节点主机名，粒度最细）
         2. topology.kubernetes.io/zone‌（可用区）
         3. topology.kubernetes.io/region‌（地域）
   
   4. 污点和容忍
   
      1. 污点
   
         1. NoSchedule
         2. PreferNoSchedule
         3. NoExecute
   
      2. 示例
   
         ```yaml
         ---
         tolerations:
         	-key: "key1"
         	operator: "equal"
         	value: "value1"
         	effect: "NoSchedule"
         ---
         tolerations:
         	-key: "key1"
         	operator: "equal"
         	value: "value1"
         	effect: "NoSchedule"
         	#多少时间内完成驱离
         	tolerationSeconds: 3600
         ---
         #只要有key1这个污点就不调度
         tolerations:
         	-key: "key1"
         	operator: "Exists"
         	effect: "NoSchedule"
         ---
         #容忍所有污点
         tolerations:
         	operator: "Exists"
         ---
         #容忍所有污点key1的污点
         tolerations:
         	-key: "key1"
         	operator: "Exists"
         ```
   
   5. 指定节点调度
   
      ```yaml
      NodeName: node01
      ```
   
   6. 指定节点标签调度
   
      ```yaml
      #强制约束，不匹配则不运行
      nodeSelector:
      	type: node01
      ```
   
      









---

附录：

1. kubectl工具
   1. create
   2. apply:没有则创建，有则不变更
   3. replace
   4. delete
   5. edit
   6. proxy：暴露apiserver到某个服务中
2. linux工具
   1. bash-completion

https://blog.csdn.net/weixin_39246554/article/details/128184637
[edit](https://github.com/wurara/wurara.github.io/new/master/_posts)
