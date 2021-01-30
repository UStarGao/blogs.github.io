---
layout: post
title: kind:ReplicationController template
---
在用户定义范围内，如果pod增多，则ReplicationController会终止额外的pod，如果减少，RC会创建新的pod，始终保持在定义范围。例如，RC会在Pod维护（例如内核升级）后在节点上重新创建新Pod。
- ReplicationController会替换由于某些原因而被删除或终止的pod，例如在节点故障或中断节点维护（例如内核升级）的情况下。因此，即使应用只需要一个pod，我们也建议使用ReplicationController
- RC跨多个Node节点监视多个pod
详情可参考中文社区：http://docs.kubernetes.org.cn/437.html

![容器映射](http://ucloudspt.cn-sh2.ufileos.com/Blogs/容器映射.png)

```
apiVersion: v1 #指定api版本，此值必须在kubectl apiversion中
kind: ReplicationController #指定创建资源的角色/类型
metadata: #资源元数据/属性
  name: test-rc #资源名字。在同一个namespace中必须唯一
  labels: #设定资源标签
  k8s-app: apache
    software: apache
    project: test
    app: test-rc
    version: v1
  annotations: #自定义注解列表
    - name: String #自定义注解名字
spec:
  replicas: 2 #副本数量2
  selector: #RC通过spec.selector来筛选要控制的pod
    software: apache
    project: test
    app: test-rc
    version: v1
    name: test-rc
  template: #这里pod定义
    metadata:
      labels: #Pod的label，这个label与spec.selector相同
        software: apache
        project: test
        app: test-rc
        version: v1
        name: test-rc
    spec: #specification of the resource content
      restartPolicy: Always #表明该容器一直运行，退出会自动重建
      nodeSelector: #节点选择
        zone: node1
      containers:
      - name: web04-pod #容器的名字
        image: web:apache #容器使用的镜像地址
        imagePullPolicy: Never #容器启动时检查镜像策略,Always每次都检查,Never从不检查(不管本地是否有),IfNotPresent本地有就不检查,如果没有就拉取
        command: ['sh'] #启动容器的运行命令,将覆盖容器中的Entrypoint,对应Dockerfile中的ENTRYPOINT
        args: ['$(str)'] #启动容器的命令参数，对应Dockerfile中的CMD参数
        env:  #指定容器中的环境变量
        - name: str #变量名称
          value: '/etc/run.sh' #变量值
        resources: #资源管理
          requests: #容器运行最低资源需求
            cpu: 0.1 #CPU资源(核数),最少值为0.001核(1m)
            memory: 32Mi #内存使用量
          limits:
            cpu: 0.5
            memory: 32Mi
        ports:
        - containerPort: 80 #容器开放对外端口
          name: httpd #名称
          protocol: TCP
        livenessProbe: #pod内容器健康检查的设置
          httpGet: #通过httpget检查健康200-399之间容器正常
          path: / #URI地址
          port: 80
          #host: 127.0.0.1 #主机地址
          scheme: HTTP
        initialDelaySeconds: 180 #表明第一次检测在容器启动后多次时间后开始
        timeoutSeconds: 5 #检测的超时时间
        periodSeconds: 15 #检查间隔时间
        #方法2
        #exec: 执行命令的发法进行监测，如果其退出码不为0，则认为容器正常
        #command:
          #- cat
          #- /tmp/health
        #方法3
        #tcpSocket: //通过tcpSocket检查健康
          #port: number
        lifecycle: #声明周期管理
          postStart: #容器创建之后，运行之前运行的任务
            exec:
              command:
                - 'sh'
                - 'yum upgrade -y'
          preStop: #容器关闭之前运行的任务
            exec:
              command: ['service httpd stop']
      volumeMounts: #永久存储挂载
      - name: volume #挂载设备名字，与volumes[*].name 需要对应
        mountPath: /data #挂载到容器的某个路径下
        readOnly: True
      volumes: #定义一组挂载设备
      - name: volume #定义一个挂载设备的名字
        #meptyDir:{}
        hostPath:
          path:/opt #挂载设备类型为hostpath，路径为宿主下的/opt
```
