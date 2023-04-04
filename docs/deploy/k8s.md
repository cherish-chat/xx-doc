# k8s部署文档

## 1. 部署前准备

### 1.1. 确保你的k8s集群已经安装了 mysql redis minio

#### 1.1.1. mysql 数据库创建

```shell
CREATE DATABASE `xxim_v1` DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

#### 1.1.2 redis 配置服务端配置

> 根据实际情况配置即可 一般来说只需修改 Redis Mysql K8sNamespace MobPush Pulsar Sms Mgmt SuperAdminPass

```json
{
  "Common": {
    "Host": "0.0.0.0",
    "RpcTimeOut": 10000,
    "LogLevel": "info",
    "Telemetry": {
      "EndPoint": "",
      "Sampler": 1,
      "Batcher": "jaeger"
    },
    "Redis": {
      "Host": "im-test-redis:6379",
      "Type": "node",
      "Pass": "testing",
      "Tls": false
    },
    "Mysql": {
      "Addr": "root:testing@tcp(im-test-mysql:3306)/xxim_v1?charset=utf8mb4&parseTime=True&loc=Local&timeout=20s&readTimeout=20s&writeTimeout=20s",
      "MaxIdleConns": 10,
      "MaxOpenConns": 100,
      "LogLevel": "error"
    },
    "Ip2RegionUrl": "https://xxim-public-1312910328.cos.ap-guangzhou.myqcloud.com/ip2region.xdb",
    "Mode": "pro"
  },
  "ConnRpc": {
    "DiscovType": "k8s",
    "K8sNamespace": "im-project",
    "Endpoints": [
      "127.0.0.1:6700"
    ],
    "Port": 6700,
    "WebsocketPort": 6701,
    "RsaPublicKey": "",
    "RsaPrivateKey": ""
  },
  "ImRpc": {
    "Port": 6702
  },
  "MsgRpc": {
    "DiscovType": "k8s",
    "K8sNamespace": "im-project",
    "Endpoints": [
      "127.0.0.1:6703"
    ],
    "Port": 6703,
    "MobPush": {
      "Enabled": false,
      "AppKey": "",
      "AppSecret": "",
      "ApnsProduction": true,
      "ApnsCateGory": "",
      "ApnsSound": "default",
      "AndroidSound": "default"
    },
    "Pulsar": {
      "Enabled": false,
      "Token": "",
      "VpcUrl": "",
      "TopicName": "",
      "ReceiverQueueSize": 1000,
      "ProducerTimeout": 3000
    }
  },
  "UserRpc": {
    "Port": 6704,
    "Sms": {
      "Enabled": false,
      "Type": "tencent",
      "TencentSms": {
        "AppId": "",
        "SecretId": "",
        "SecretKey": "",
        "Region": "",
        "Sign": "",
        "TemplateId": ""
      }
    }
  },
  "RelationRpc": {
    "Port": 6705
  },
  "GroupRpc": {
    "Port": 6706,
    "MaxGroupCount": 2000,
    "MaxGroupMemberCount": 200000
  },
  "NoticeRpc": {
    "Port": 6707
  },
  "AppMgmtRpc": {
    "Port": 6709
  },
  "Mgmt": {
    "RpcPort": 6708,
    "HttpPort": 6799,
    "SuperAdminId": "superadmin",
    "SuperAdminPass": "superadmin"
  },
  "Xos": {
    "HttpPort": 6800
  }
}
```

```shell
set s:model:server_config $serverConfig
```

#### 1.1.3 minio开启download模式

```go
package main

import (
	"context"
	minio "github.com/minio/minio-go/v7"
	"github.com/minio/minio-go/v7/pkg/credentials"
)

type MinioConfig struct {
	Endpoint        string `protobuf:"bytes,1,opt,name=endpoint,proto3" json:"endpoint"`
	AccessKeyId     string `protobuf:"bytes,2,opt,name=accessKeyId,proto3" json:"accessKeyId"`
	SecretAccessKey string `protobuf:"bytes,3,opt,name=secretAccessKey,proto3" json:"secretAccessKey"`
	BucketName      string `protobuf:"bytes,4,opt,name=bucketName,proto3" json:"bucketName"`
	Ssl             bool   `protobuf:"varint,5,opt,name=ssl,proto3" json:"ssl"`
	BucketUrl       string `protobuf:"bytes,6,opt,name=bucketUrl,proto3" json:"bucketUrl"`
	Region          string `protobuf:"bytes,7,opt,name=region,proto3" json:"region"`
}

// MinioStorage minio存储 实现Storage接口
type MinioStorage struct {
	Config *MinioConfig
	Client *minio.Client
}

func (s *MinioStorage) ExistObject(ctx context.Context, key string) (exists bool, err error) {
	_, err = s.Client.StatObject(ctx, s.Config.BucketName, key, minio.StatObjectOptions{})
	if err != nil {
		e, ok := err.(minio.ErrorResponse)
		if ok && e.Code == "NoSuchKey" {
			return false, nil
		}
		return false, err
	}
	return true, nil
}

// NewMinioStorage 创建minio存储
func NewMinioStorage(config *MinioConfig) (*MinioStorage, error) {
	client, err := minio.New(config.Endpoint, &minio.Options{
		Creds:  credentials.NewStaticV4(config.AccessKeyId, config.SecretAccessKey, ""),
		Secure: config.Ssl,
		Region: config.Region,
	})
	if err != nil {
		return nil, err
	}
	s := &MinioStorage{
		Config: config,
		Client: client,
	}
	return s, nil
}

func (s *MinioStorage) setDownloadPolicy() error {
	// mc policy set download xxim
	err := s.Client.SetBucketPolicy(context.Background(), s.Config.BucketName, `{"Version":"2012-10-17","Statement":[{"Effect":"Allow","Principal":{"AWS":["*"]},"Action":["s3:GetObject"],"Resource":["arn:aws:s3:::`+s.Config.BucketName+`/*"]}]}`)
	if err != nil {
		return err
	}
	return nil
}

func main() {
	storage, err := NewMinioStorage(&MinioConfig{
		Endpoint:        "YOURENDPOINT",
		AccessKeyId:     "YOURACCESSKEY",
		SecretAccessKey: "YOURSECRETKEY",
		BucketName:      "xxim",
		Ssl:             false,
		BucketUrl:       "http://YOURENDPOINT/xxim",
		Region:          "",
	})
	if err != nil {
		panic(err)
	}
	err = storage.setDownloadPolicy()
	if err != nil {
		panic(err)
	}
}
```

## 2. 部署

### 2.1 serviceAccount 服务账户

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: find-endpoints
```

### 2.2. clusterRole 集群角色

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: find-endpoints
rules:
  - apiGroups: [ "" ]
    resources: [ "endpoints" ]
    verbs: [ "get", "list", "watch" ]
```

### 2.3. clusterRoleBinding 集群角色绑定

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: find-endpoints
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: find-endpoints
subjects:
  - kind: ServiceAccount
    name: find-endpoints
```

### 2.4. configMap 配置文件

```yaml
apiVersion: v1
data:
  DEBUG: "1"
kind: ConfigMap
metadata:
  name: xxim-all-config
```

### 2.5. service 服务

#### 2.5.1. mgmt-svc

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mgmt-svc
spec:
  ports:
    - name: port-1
      port: 6708
      protocol: TCP
      targetPort: 6708
    - name: port-2
      port: 6799
      protocol: TCP
      targetPort: 6799
  selector:
    app: mgmt
  type: ClusterIP
```

#### 2.5.2. appmgmt-rpc-svc

```yaml
apiVersion: v1
kind: Service
metadata:
  name: appmgmt-rpc-svc
spec:
  ports:
    - name: port-1
      port: 6709
      protocol: TCP
      targetPort: 6709
  selector:
    app: appmgmt-rpc
  type: ClusterIP
```

#### 2.5.3. conn-rpc-svc

```yaml
apiVersion: v1
kind: Service
metadata:
  name: conn-rpc-svc
spec:
  ports:
    - name: port-1
      port: 6700
      protocol: TCP
      targetPort: 6700
    - name: port-2
      port: 6701
      protocol: TCP
      targetPort: 6701
  selector:
    app: conn-rpc
  type: ClusterIP
```

#### 2.5.4. group-rpc-svc

```yaml
apiVersion: v1
kind: Service
metadata:
  name: group-rpc-svc
spec:
  ports:
    - name: port-1
      port: 6706
      protocol: TCP
      targetPort: 6706
  selector:
    app: group-rpc
  type: ClusterIP
```

#### 2.5.5. im-rpc-svc

```yaml
apiVersion: v1
kind: Service
metadata:
  name: im-rpc-svc
spec:
  ports:
    - name: port-1
      port: 6702
      protocol: TCP
      targetPort: 6702
  selector:
    app: im-rpc
  type: ClusterIP
```

#### 2.5.6. msg-rpc-svc

```yaml
apiVersion: v1
kind: Service
metadata:
  name: msg-rpc-svc
spec:
  ports:
    - name: port-1
      port: 6703
      protocol: TCP
      targetPort: 6703
  selector:
    app: msg-rpc
  type: ClusterIP
```

#### 2.5.7. notice-rpc-svc

```yaml
apiVersion: v1
kind: Service
metadata:
  name: notice-rpc-svc
spec:
  ports:
    - name: port-1
      port: 6707
      protocol: TCP
      targetPort: 6707
  selector:
    app: notice-rpc
  type: ClusterIP
```

#### 2.5.8. relation-rpc-svc

```yaml
apiVersion: v1
kind: Service
metadata:
  name: relation-rpc-svc
spec:
  ports:
    - name: port-1
      port: 6705
      protocol: TCP
      targetPort: 6705
  selector:
    app: relation-rpc
  type: ClusterIP
```

#### 2.5.9. user-rpc-svc

```yaml
apiVersion: v1
kind: Service
metadata:
  name: user-rpc-svc
spec:
  ports:
    - name: port-1
      port: 6704
      protocol: TCP
      targetPort: 6704
  selector:
    app: user-rpc
  type: ClusterIP
```

#### 2.5.10. xos-svc

```yaml
apiVersion: v1
kind: Service
metadata:
  name: xos-svc
spec:
  ports:
    - name: port-1
      port: 6800
      protocol: TCP
      targetPort: 6800
  selector:
    app: xos-api
  type: NodePort
```

### 2.6. deployment 部署

#### 2.6.1. mgmt

> 需要注意启动命令中包含 redis的地址和密码

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mgmt
spec:
  progressDeadlineSeconds: 600
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: mgmt
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: mgmt
      name: mgmt
    spec:
      containers:
        - command:
            - bash
            - -c
            - ./bin -host 10.1.4.3:6379 -pass '' -type node
          env:
            - name: NODE_NAME
              valueFrom:
                fieldRef:
                  apiVersion: v1
                  fieldPath: spec.nodeName
            - name: POD_NAME
              valueFrom:
                fieldRef:
                  apiVersion: v1
                  fieldPath: metadata.name
            - name: NODE_IP
              valueFrom:
                fieldRef:
                  apiVersion: v1
                  fieldPath: status.hostIP
            - name: POD_IP
              valueFrom:
                fieldRef:
                  apiVersion: v1
                  fieldPath: status.podIP
          envFrom:
            - configMapRef:
                name: xxim-all-config
          image: registry.cn-shanghai.aliyuncs.com/xxim-dev/xxim-mgmt-rpc:20230313134251
          imagePullPolicy: IfNotPresent
          lifecycle:
            preStop:
              exec:
                command:
                  - sh
                  - -c
                  - sleep 5
          livenessProbe:
            failureThreshold: 3
            initialDelaySeconds: 15
            periodSeconds: 20
            successThreshold: 1
            tcpSocket:
              port: 6708
            timeoutSeconds: 1
          name: mgmt
          ports:
            - containerPort: 6708
              protocol: TCP
          readinessProbe:
            failureThreshold: 3
            initialDelaySeconds: 3
            periodSeconds: 10
            successThreshold: 1
            tcpSocket:
              port: 6708
            timeoutSeconds: 1
          resources: { }
          securityContext:
            privileged: false
          terminationMessagePath: /dev/termination-log
          terminationMessagePolicy: File
          volumeMounts:
            - mountPath: /etc/localtime
              name: timezone
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: { }
      serviceAccount: find-endpoints
      serviceAccountName: find-endpoints
      terminationGracePeriodSeconds: 30
      volumes:
        - hostPath:
            path: /usr/share/zoneinfo/Asia/Shanghai
            type: ""
          name: timezone
```

#### 2.6.2. appmgmt-rpc

```yaml
kind: Deployment
apiVersion: apps/v1
metadata:
  name: appmgmt-rpc
spec:
  replicas: 1
  selector:
    matchLabels:
      app: appmgmt-rpc
  template:
    metadata:
      name: appmgmt-rpc
      labels:
        app: appmgmt-rpc
    spec:
      volumes:
        - name: timezone
          hostPath:
            path: /usr/share/zoneinfo/Asia/Shanghai
            type: ''
      containers:
        - name: appmgmt-rpc
          image: >-
            registry.cn-shanghai.aliyuncs.com/xxim-dev/xxim-appmgmt-rpc:20230305183603
          command:
            - ./bin
            - '-a'
            - 'mgmt-svc:6708'
          ports:
            - containerPort: 6709
              protocol: TCP
          envFrom:
            - configMapRef:
                name: xxim-all-config
          env:
            - name: NODE_NAME
              valueFrom:
                fieldRef:
                  apiVersion: v1
                  fieldPath: spec.nodeName
            - name: POD_NAME
              valueFrom:
                fieldRef:
                  apiVersion: v1
                  fieldPath: metadata.name
            - name: NODE_IP
              valueFrom:
                fieldRef:
                  apiVersion: v1
                  fieldPath: status.hostIP
            - name: POD_IP
              valueFrom:
                fieldRef:
                  apiVersion: v1
                  fieldPath: status.podIP
          resources: { }
          volumeMounts:
            - name: timezone
              mountPath: /etc/localtime
          livenessProbe:
            tcpSocket:
              port: 6709
            initialDelaySeconds: 15
            timeoutSeconds: 1
            periodSeconds: 20
            successThreshold: 1
            failureThreshold: 3
          readinessProbe:
            tcpSocket:
              port: 6709
            initialDelaySeconds: 3
            timeoutSeconds: 1
            periodSeconds: 10
            successThreshold: 1
            failureThreshold: 3
          lifecycle:
            preStop:
              exec:
                command:
                  - sh
                  - '-c'
                  - sleep 5
          terminationMessagePath: /dev/termination-log
          terminationMessagePolicy: File
          imagePullPolicy: IfNotPresent
          securityContext:
            privileged: false
      restartPolicy: Always
      terminationGracePeriodSeconds: 30
      dnsPolicy: ClusterFirst
      serviceAccountName: find-endpoints
      serviceAccount: find-endpoints
      securityContext: { }
      schedulerName: default-scheduler
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 25%
      maxSurge: 25%
  revisionHistoryLimit: 10
  progressDeadlineSeconds: 600
```

#### 2.6.3. conn-rpc

```yaml
kind: Deployment
apiVersion: apps/v1
metadata:
  name: conn-rpc
spec:
  replicas: 1
  selector:
    matchLabels:
      app: conn-rpc
  template:
    metadata:
      name: conn-rpc
      labels:
        app: conn-rpc
    spec:
      volumes:
        - name: timezone
          hostPath:
            path: /usr/share/zoneinfo/Asia/Shanghai
            type: ''
      containers:
        - name: conn-rpc
          image: >-
            registry.cn-shanghai.aliyuncs.com/xxim-dev/xxim-conn-rpc:20230312162816
          command:
            - ./bin
            - '-a'
            - 'mgmt-svc:6708'
          ports:
            - containerPort: 6700
              protocol: TCP
          envFrom:
            - configMapRef:
                name: xxim-all-config
          env:
            - name: NODE_NAME
              valueFrom:
                fieldRef:
                  apiVersion: v1
                  fieldPath: spec.nodeName
            - name: POD_NAME
              valueFrom:
                fieldRef:
                  apiVersion: v1
                  fieldPath: metadata.name
            - name: NODE_IP
              valueFrom:
                fieldRef:
                  apiVersion: v1
                  fieldPath: status.hostIP
            - name: POD_IP
              valueFrom:
                fieldRef:
                  apiVersion: v1
                  fieldPath: status.podIP
          resources: { }
          volumeMounts:
            - name: timezone
              mountPath: /etc/localtime
          livenessProbe:
            tcpSocket:
              port: 6700
            initialDelaySeconds: 15
            timeoutSeconds: 1
            periodSeconds: 20
            successThreshold: 1
            failureThreshold: 3
          readinessProbe:
            tcpSocket:
              port: 6700
            initialDelaySeconds: 3
            timeoutSeconds: 1
            periodSeconds: 10
            successThreshold: 1
            failureThreshold: 3
          lifecycle:
            preStop:
              exec:
                command:
                  - sh
                  - '-c'
                  - sleep 5
          terminationMessagePath: /dev/termination-log
          terminationMessagePolicy: File
          imagePullPolicy: IfNotPresent
          securityContext:
            privileged: false
      restartPolicy: Always
      terminationGracePeriodSeconds: 30
      dnsPolicy: ClusterFirst
      serviceAccountName: find-endpoints
      serviceAccount: find-endpoints
      securityContext: { }
      schedulerName: default-scheduler
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 25%
      maxSurge: 25%
  revisionHistoryLimit: 10
  progressDeadlineSeconds: 600
```

#### 2.6.4. msg-rpc

```yaml
kind: Deployment
apiVersion: apps/v1
metadata:
  name: msg-rpc
spec:
  replicas: 1
  selector:
    matchLabels:
      app: msg-rpc
  template:
    metadata:
      name: msg-rpc
      labels:
        app: msg-rpc
    spec:
      volumes:
        - name: timezone
          hostPath:
            path: /usr/share/zoneinfo/Asia/Shanghai
            type: ''
      containers:
        - name: msg-rpc
          image: >-
            registry.cn-shanghai.aliyuncs.com/xxim-dev/xxim-msg-rpc:20230304213107
          command:
            - ./bin
            - '-a'
            - 'mgmt-svc:6708'
          ports:
            - containerPort: 6703
              protocol: TCP
          envFrom:
            - configMapRef:
                name: xxim-all-config
          env:
            - name: NODE_NAME
              valueFrom:
                fieldRef:
                  apiVersion: v1
                  fieldPath: spec.nodeName
            - name: POD_NAME
              valueFrom:
                fieldRef:
                  apiVersion: v1
                  fieldPath: metadata.name
            - name: NODE_IP
              valueFrom:
                fieldRef:
                  apiVersion: v1
                  fieldPath: status.hostIP
            - name: POD_IP
              valueFrom:
                fieldRef:
                  apiVersion: v1
                  fieldPath: status.podIP
          resources: { }
          volumeMounts:
            - name: timezone
              mountPath: /etc/localtime
          livenessProbe:
            tcpSocket:
              port: 6703
            initialDelaySeconds: 15
            timeoutSeconds: 1
            periodSeconds: 20
            successThreshold: 1
            failureThreshold: 3
          readinessProbe:
            tcpSocket:
              port: 6703
            initialDelaySeconds: 3
            timeoutSeconds: 1
            periodSeconds: 10
            successThreshold: 1
            failureThreshold: 3
          lifecycle:
            preStop:
              exec:
                command:
                  - sh
                  - '-c'
                  - sleep 5
          terminationMessagePath: /dev/termination-log
          terminationMessagePolicy: File
          imagePullPolicy: IfNotPresent
          securityContext:
            privileged: false
      restartPolicy: Always
      terminationGracePeriodSeconds: 30
      dnsPolicy: ClusterFirst
      serviceAccountName: find-endpoints
      serviceAccount: find-endpoints
      securityContext: { }
      schedulerName: default-scheduler
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 25%
      maxSurge: 25%
  revisionHistoryLimit: 10
  progressDeadlineSeconds: 600
```

#### 2.6.5. user-rpc

```yaml
kind: Deployment
apiVersion: apps/v1
metadata:
  name: user-rpc
spec:
  replicas: 5
  selector:
    matchLabels:
      app: user-rpc
  template:
    metadata:
      name: user-rpc
      labels:
        app: user-rpc
    spec:
      volumes:
        - name: timezone
          hostPath:
            path: /usr/share/zoneinfo/Asia/Shanghai
            type: ''
      containers:
        - name: user-rpc
          image: >-
            registry.cn-shanghai.aliyuncs.com/xxim-dev/xxim-user-rpc:20230307165647
          command:
            - ./bin
            - '-a'
            - 'mgmt-svc:6708'
          ports:
            - containerPort: 6704
              protocol: TCP
          envFrom:
            - configMapRef:
                name: xxim-all-config
          env:
            - name: NODE_NAME
              valueFrom:
                fieldRef:
                  apiVersion: v1
                  fieldPath: spec.nodeName
            - name: POD_NAME
              valueFrom:
                fieldRef:
                  apiVersion: v1
                  fieldPath: metadata.name
            - name: NODE_IP
              valueFrom:
                fieldRef:
                  apiVersion: v1
                  fieldPath: status.hostIP
            - name: POD_IP
              valueFrom:
                fieldRef:
                  apiVersion: v1
                  fieldPath: status.podIP
          resources: { }
          volumeMounts:
            - name: timezone
              mountPath: /etc/localtime
          livenessProbe:
            tcpSocket:
              port: 6704
            initialDelaySeconds: 15
            timeoutSeconds: 1
            periodSeconds: 20
            successThreshold: 1
            failureThreshold: 3
          readinessProbe:
            tcpSocket:
              port: 6704
            initialDelaySeconds: 3
            timeoutSeconds: 1
            periodSeconds: 10
            successThreshold: 1
            failureThreshold: 3
          lifecycle:
            preStop:
              exec:
                command:
                  - sh
                  - '-c'
                  - sleep 5
          terminationMessagePath: /dev/termination-log
          terminationMessagePolicy: File
          imagePullPolicy: IfNotPresent
          securityContext:
            privileged: false
      restartPolicy: Always
      terminationGracePeriodSeconds: 30
      dnsPolicy: ClusterFirst
      serviceAccountName: find-endpoints
      serviceAccount: find-endpoints
      securityContext: { }
      imagePullSecrets:
        - name: tencent-xxim
      schedulerName: default-scheduler
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 25%
      maxSurge: 25%
  revisionHistoryLimit: 10
  progressDeadlineSeconds: 600
```

#### 2.6.6. im-rpc

```yaml
kind: Deployment
apiVersion: apps/v1
metadata:
  name: im-rpc
spec:
  replicas: 1
  selector:
    matchLabels:
      app: im-rpc
  template:
    metadata:
      name: im-rpc
      labels:
        app: im-rpc
    spec:
      volumes:
        - name: timezone
          hostPath:
            path: /usr/share/zoneinfo/Asia/Shanghai
            type: ''
      containers:
        - name: im-rpc
          image: >-
            registry.cn-shanghai.aliyuncs.com/xxim-dev/xxim-im-rpc:20230304160437
          command:
            - ./bin
            - '-a'
            - 'mgmt-svc:6708'
          ports:
            - containerPort: 6702
              protocol: TCP
          envFrom:
            - configMapRef:
                name: xxim-all-config
          env:
            - name: NODE_NAME
              valueFrom:
                fieldRef:
                  apiVersion: v1
                  fieldPath: spec.nodeName
            - name: POD_NAME
              valueFrom:
                fieldRef:
                  apiVersion: v1
                  fieldPath: metadata.name
            - name: NODE_IP
              valueFrom:
                fieldRef:
                  apiVersion: v1
                  fieldPath: status.hostIP
            - name: POD_IP
              valueFrom:
                fieldRef:
                  apiVersion: v1
                  fieldPath: status.podIP
          resources: { }
          volumeMounts:
            - name: timezone
              mountPath: /etc/localtime
          livenessProbe:
            tcpSocket:
              port: 6702
            initialDelaySeconds: 15
            timeoutSeconds: 1
            periodSeconds: 20
            successThreshold: 1
            failureThreshold: 3
          readinessProbe:
            tcpSocket:
              port: 6702
            initialDelaySeconds: 3
            timeoutSeconds: 1
            periodSeconds: 10
            successThreshold: 1
            failureThreshold: 3
          lifecycle:
            preStop:
              exec:
                command:
                  - sh
                  - '-c'
                  - sleep 5
          terminationMessagePath: /dev/termination-log
          terminationMessagePolicy: File
          imagePullPolicy: IfNotPresent
          securityContext:
            privileged: false
      restartPolicy: Always
      terminationGracePeriodSeconds: 30
      dnsPolicy: ClusterFirst
      serviceAccountName: find-endpoints
      serviceAccount: find-endpoints
      securityContext: { }
      schedulerName: default-scheduler
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 25%
      maxSurge: 25%
  revisionHistoryLimit: 10
  progressDeadlineSeconds: 600
```

#### 2.6.7. group-rpc

```yaml
kind: Deployment
apiVersion: apps/v1
metadata:
  name: group-rpc
spec:
  replicas: 1
  selector:
    matchLabels:
      app: group-rpc
  template:
    metadata:
      name: group-rpc
      labels:
        app: group-rpc
    spec:
      volumes:
        - name: timezone
          hostPath:
            path: /usr/share/zoneinfo/Asia/Shanghai
            type: ''
      containers:
        - name: group-rpc
          image: >-
            registry.cn-shanghai.aliyuncs.com/xxim-dev/xxim-group-rpc:20230310222141
          command:
            - ./bin
            - '-a'
            - 'mgmt-svc:6708'
          ports:
            - containerPort: 6706
              protocol: TCP
          envFrom:
            - configMapRef:
                name: xxim-all-config
          env:
            - name: NODE_NAME
              valueFrom:
                fieldRef:
                  apiVersion: v1
                  fieldPath: spec.nodeName
            - name: POD_NAME
              valueFrom:
                fieldRef:
                  apiVersion: v1
                  fieldPath: metadata.name
            - name: NODE_IP
              valueFrom:
                fieldRef:
                  apiVersion: v1
                  fieldPath: status.hostIP
            - name: POD_IP
              valueFrom:
                fieldRef:
                  apiVersion: v1
                  fieldPath: status.podIP
          resources: { }
          volumeMounts:
            - name: timezone
              mountPath: /etc/localtime
          livenessProbe:
            tcpSocket:
              port: 6706
            initialDelaySeconds: 15
            timeoutSeconds: 1
            periodSeconds: 20
            successThreshold: 1
            failureThreshold: 3
          readinessProbe:
            tcpSocket:
              port: 6706
            initialDelaySeconds: 3
            timeoutSeconds: 1
            periodSeconds: 10
            successThreshold: 1
            failureThreshold: 3
          lifecycle:
            preStop:
              exec:
                command:
                  - sh
                  - '-c'
                  - sleep 5
          terminationMessagePath: /dev/termination-log
          terminationMessagePolicy: File
          imagePullPolicy: IfNotPresent
          securityContext:
            privileged: false
      restartPolicy: Always
      terminationGracePeriodSeconds: 30
      dnsPolicy: ClusterFirst
      serviceAccountName: find-endpoints
      serviceAccount: find-endpoints
      securityContext: { }
      schedulerName: default-scheduler
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 25%
      maxSurge: 25%
  revisionHistoryLimit: 10
  progressDeadlineSeconds: 600
```

#### 2.6.8. relation-rpc

```yaml
kind: Deployment
apiVersion: apps/v1
metadata:
  name: relation-rpc
  namespace: im-project
spec:
  replicas: 1
  selector:
    matchLabels:
      app: relation-rpc
  template:
    metadata:
      name: relation-rpc
      labels:
        app: relation-rpc
    spec:
      volumes:
        - name: timezone
          hostPath:
            path: /usr/share/zoneinfo/Asia/Shanghai
            type: ''
      containers:
        - name: relation-rpc
          image: >-
            registry.cn-shanghai.aliyuncs.com/xxim-dev/xxim-relation-rpc:20230312162816
          command:
            - ./bin
            - '-a'
            - 'mgmt-svc:6708'
          ports:
            - containerPort: 6705
              protocol: TCP
          envFrom:
            - configMapRef:
                name: xxim-all-config
          env:
            - name: NODE_NAME
              valueFrom:
                fieldRef:
                  apiVersion: v1
                  fieldPath: spec.nodeName
            - name: POD_NAME
              valueFrom:
                fieldRef:
                  apiVersion: v1
                  fieldPath: metadata.name
            - name: NODE_IP
              valueFrom:
                fieldRef:
                  apiVersion: v1
                  fieldPath: status.hostIP
            - name: POD_IP
              valueFrom:
                fieldRef:
                  apiVersion: v1
                  fieldPath: status.podIP
          resources: { }
          volumeMounts:
            - name: timezone
              mountPath: /etc/localtime
          livenessProbe:
            tcpSocket:
              port: 6705
            initialDelaySeconds: 15
            timeoutSeconds: 1
            periodSeconds: 20
            successThreshold: 1
            failureThreshold: 3
          readinessProbe:
            tcpSocket:
              port: 6705
            initialDelaySeconds: 3
            timeoutSeconds: 1
            periodSeconds: 10
            successThreshold: 1
            failureThreshold: 3
          lifecycle:
            preStop:
              exec:
                command:
                  - sh
                  - '-c'
                  - sleep 5
          terminationMessagePath: /dev/termination-log
          terminationMessagePolicy: File
          imagePullPolicy: IfNotPresent
          securityContext:
            privileged: false
      restartPolicy: Always
      terminationGracePeriodSeconds: 30
      dnsPolicy: ClusterFirst
      serviceAccountName: find-endpoints
      serviceAccount: find-endpoints
      securityContext: { }
      schedulerName: default-scheduler
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 25%
      maxSurge: 25%
  revisionHistoryLimit: 10
  progressDeadlineSeconds: 600
```

#### 2.6.8. notice-rpc

```yaml
kind: Deployment
apiVersion: apps/v1
metadata:
  name: notice-rpc
spec:
  replicas: 1
  selector:
    matchLabels:
      app: notice-rpc
  template:
    metadata:
      name: notice-rpc
      labels:
        app: notice-rpc
    spec:
      volumes:
        - name: timezone
          hostPath:
            path: /usr/share/zoneinfo/Asia/Shanghai
            type: ''
      containers:
        - name: notice-rpc
          image: >-
            registry.cn-shanghai.aliyuncs.com/xxim-dev/xxim-notice-rpc:20230304160141
          command:
            - ./bin
            - '-a'
            - 'mgmt-svc:6708'
          ports:
            - containerPort: 6707
              protocol: TCP
          envFrom:
            - configMapRef:
                name: xxim-all-config
          env:
            - name: NODE_NAME
              valueFrom:
                fieldRef:
                  apiVersion: v1
                  fieldPath: spec.nodeName
            - name: POD_NAME
              valueFrom:
                fieldRef:
                  apiVersion: v1
                  fieldPath: metadata.name
            - name: NODE_IP
              valueFrom:
                fieldRef:
                  apiVersion: v1
                  fieldPath: status.hostIP
            - name: POD_IP
              valueFrom:
                fieldRef:
                  apiVersion: v1
                  fieldPath: status.podIP
          resources: { }
          volumeMounts:
            - name: timezone
              mountPath: /etc/localtime
          livenessProbe:
            tcpSocket:
              port: 6707
            initialDelaySeconds: 15
            timeoutSeconds: 1
            periodSeconds: 20
            successThreshold: 1
            failureThreshold: 3
          readinessProbe:
            tcpSocket:
              port: 6707
            initialDelaySeconds: 3
            timeoutSeconds: 1
            periodSeconds: 10
            successThreshold: 1
            failureThreshold: 3
          lifecycle:
            preStop:
              exec:
                command:
                  - sh
                  - '-c'
                  - sleep 5
          terminationMessagePath: /dev/termination-log
          terminationMessagePolicy: File
          imagePullPolicy: IfNotPresent
          securityContext:
            privileged: false
      restartPolicy: Always
      terminationGracePeriodSeconds: 30
      dnsPolicy: ClusterFirst
      serviceAccountName: find-endpoints
      serviceAccount: find-endpoints
      securityContext: { }
      schedulerName: default-scheduler
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 25%
      maxSurge: 25%
  revisionHistoryLimit: 10
  progressDeadlineSeconds: 600
```

#### 2.6.9. xos-api

```yaml
kind: Deployment
apiVersion: apps/v1
metadata:
  name: xos-api
spec:
  replicas: 1
  selector:
    matchLabels:
      app: xos-api
  template:
    metadata:
      name: xos-api
      labels:
        app: xos-api
    spec:
      volumes:
        - name: timezone
          hostPath:
            path: /usr/share/zoneinfo/Asia/Shanghai
            type: ''
      containers:
        - name: xos-api
          image: >-
            registry.cn-shanghai.aliyuncs.com/xxim-dev/xxim-xos-api:20230405004944
          command:
            - ./bin
            - '-a'
            - 'mgmt-svc:6708'
          ports:
            - containerPort: 6800
              protocol: TCP
          envFrom:
            - configMapRef:
                name: xxim-all-config
          env:
            - name: NODE_NAME
              valueFrom:
                fieldRef:
                  apiVersion: v1
                  fieldPath: spec.nodeName
            - name: POD_NAME
              valueFrom:
                fieldRef:
                  apiVersion: v1
                  fieldPath: metadata.name
            - name: NODE_IP
              valueFrom:
                fieldRef:
                  apiVersion: v1
                  fieldPath: status.hostIP
            - name: POD_IP
              valueFrom:
                fieldRef:
                  apiVersion: v1
                  fieldPath: status.podIP
          resources: { }
          volumeMounts:
            - name: timezone
              mountPath: /etc/localtime
          livenessProbe:
            tcpSocket:
              port: 6800
            initialDelaySeconds: 15
            timeoutSeconds: 1
            periodSeconds: 20
            successThreshold: 1
            failureThreshold: 3
          readinessProbe:
            tcpSocket:
              port: 6800
            initialDelaySeconds: 3
            timeoutSeconds: 1
            periodSeconds: 10
            successThreshold: 1
            failureThreshold: 3
          lifecycle:
            preStop:
              exec:
                command:
                  - sh
                  - '-c'
                  - sleep 5
          terminationMessagePath: /dev/termination-log
          terminationMessagePolicy: File
          imagePullPolicy: IfNotPresent
          securityContext:
            privileged: false
      restartPolicy: Always
      terminationGracePeriodSeconds: 30
      dnsPolicy: ClusterFirst
      serviceAccountName: find-endpoints
      serviceAccount: find-endpoints
      securityContext: { }
      schedulerName: default-scheduler
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 25%
      maxSurge: 25%
  revisionHistoryLimit: 10
  progressDeadlineSeconds: 600
```

### 2.7. 检查应用镜像tag是否为最新

| 应用名称         | 镜像tag          | 镜像地址                                                         |
|--------------|----------------|--------------------------------------------------------------|
| appmgmt-rpc  | 20230404231311 | registry.cn-shanghai.aliyuncs.com/xxim-dev/xxim-appmgmt-rpc  |
| conn-rpc     | 20230405015304 | registry.cn-shanghai.aliyuncs.com/xxim-dev/xxim-conn-rpc     |
| group-rpc    | 20230324175110 | registry.cn-shanghai.aliyuncs.com/xxim-dev/xxim-group-rpc    |
| im-rpc       | 20230320010459 | registry.cn-shanghai.aliyuncs.com/xxim-dev/xxim-im-rpc       |
| mgmt         | 20230405000146 | registry.cn-shanghai.aliyuncs.com/xxim-dev/xxim-mgmt-rpc     |
| msg-rpc      | 20230324175110 | registry.cn-shanghai.aliyuncs.com/xxim-dev/xxim-msg-rpc      |
| notice-rpc   | 20230314011313 | registry.cn-shanghai.aliyuncs.com/xxim-dev/xxim-notice-rpc   |
| relation-rpc | 20230324175110 | registry.cn-shanghai.aliyuncs.com/xxim-dev/xxim-relation-rpc |
| user-rpc     | 20230405020012 | registry.cn-shanghai.aliyuncs.com/xxim-dev/xxim-user-rpc     |
| xos-api      | 20230405004944 | registry.cn-shanghai.aliyuncs.com/xxim-dev/xxim-xos-api      |
