---
layout: post
title: kind:pod template
---
在Docker中，容器是最小的处理单元，增删改查的对象是容器，容器是一种虚拟化技术，容器之间是隔离的，隔离是基于Linux Namespace 实现的。而在Kubernetes中，Pod包含一个或者多个相关的容器，Pod可以认为是容器的一种延伸扩展，一个Pod也是一个隔离体，而Pod内部包含的一组容器又是共享的（包括PID、Network、IPC、UTS）除此之外，Pod中的容器可以访问共同的数据卷来实现文件系统的共享。

```
apiVersion: v1 #指定api版本，此值必须在kubectl apiversion中
kind: Pod #指定创建资源的角色/类型
metadata: #资源的元数据/属性
  name: web01-pod #资源名字，在同一个namespace中必须唯一
  labels: #设定资源的标签
    k8s-app: apache
    version: v1
    kubernetes.io/cluster-service: "true"
  annotainons:  #自定义注解列表
    - name: String #自定义注解名字
spec: #specification of the resource content 指定该资源的内容
  restartPolicy:  Always #表明该容器一直运行，默认k8s策略，在此容器退出后,会立即创建一个相同的容器
  nodeSelector: #节点选择
    zone: node1
  containers:
  - name: web01-pod #容器的名字
    image: web:apache #容器使用的镜像地址
    imagePullPolicy: Never #容器启动时检查仓库的策略，Always每次都检查，Never从不检查(不管本地是否有)，IfNotPresent，如果本地有就不检查，没有就拉取
    command: ['sh'] #启动容器的运行命令，将覆盖容器的Entrypoint，对应Dockefile中的ENTRYPOINT
    args: ["$(str)"] #启动容器的命令参数,对应Dockerfile中的CMD参数
    env: #指定容器中的环境变量
    - name: str #变量的名字
      value: "/etc/run.sh" #变量的值
    resources: #资源管理
      requests: #容器运行时的最低资源需求
        cpu: 0.1 #CPU资源(核数)，最少值为0.001核(1m)
        memory: 32Mi #内存资源
      limits: #资源限制
        cpu: 0.5
        memory: 32Mi
    ports:
    - containerPort: 80 #容器开放对外的端口
      name: httpd #名称
      protocol: TCP
    livenessProbe: #pod内容器健康检查的设置
      httpGet:  #通过httpget检查健康，200-399之间的容器正常
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
