# websocket gateway

## 连接

### 路径以及参数

- 路径：/ws?token=${token}&userId=${userId}&networkUsed=${networkUsed}&platform=${platform}&deviceId=${deviceId}&deviceModel=${deviceModel}&appVersion=${appVersion}&language=${language}
- 参数：

| 参数名         | 类型     | 是否必须 | 说明      | 示例          |
|:------------|:-------|:-----|:--------|:------------|
| token       | string | 是    | 用户token | ey.x.x      |
| userId      | string | 是    | 用户id    | 123456      |
| networkUsed | string | 是    | 网络类型    | wifi        |
| platform    | string | 是    | 平台类型    | ios         |
| deviceId    | string | 是    | 设备id    | xxx-xxx-xxx |
| deviceModel | string | 是    | 设备型号    | iphone 6    |
| osVersion   | string | 是    | 系统版本    | 10.3.1      |
| appVersion  | string | 是    | app版本   | 1.0.0       |
| language    | string | 是    | 语言      | zh          |

### 连接说明

    连接成功后，服务端会发送一条 `text` 类型消息，内容为 `connected`，表示连接成功。

    socket关闭原因: 
        - code=3000 表示认证失败 客户端应跳转到登录页面  msg=错误信息


### 部署说明

    如果使用云平台提供的负载均衡，需要配置支持websocket，一般云平台支持websocket最大连接时间为3600s，超过这个时间，连接会断开，需要重新连接。

### 开发说明

    如果需要多地区部署，请修改 `internal/handler/handler.go` 中的 `// 自定义负载均衡` 处代码，实现自己的负载均衡策略。

## 请求和响应

### 请求

```protobuf
// 客户端发送事件
enum ActiveEvent {
  // 发送消息
  SendMsgList = 0;
  // 批量获取会话的消息seq
  SyncConvSeq = 1;
  // 批量同步消息列表
  SyncMsgList = 2;
  // 确认消费通知
  AckNotice = 3;
  // 获取一条消息
  GetMsgById = 4;
  // 自定义请求
  CustomRequest = 100;
}

// 客户端发送的消息体
message RequestBody {
  ActiveEvent event = 1;
  string reqId = 2;
  bytes data = 3;
}
```

> 说明：客户端发送的消息体，其中 `event` 表示事件类型，`reqId` 表示请求id，`data` 表示请求数据。

#### RequestBody说明

- event: 事件类型
- reqId: 请求id，用于标识请求，服务端会原样返回
- data: 请求数据

#### RequestBody.data说明

##### SendMsgList

```protobuf
// 发送消息请求 可批量
message SendMsgListReq {
  repeated MsgData msgDataList = 2;
  // options
  // 1. 延迟时间（秒） 不得大于 864000秒 也就是10天
  optional int32 deliverAfter = 11;
}
```

##### SyncConvSeq

```protobuf
// 同步会话消息seq
message BatchGetConvSeqReq {
  repeated string convIdList = 2;
}
```

##### SyncMsgList

```protobuf
// 同步消息
message BatchGetMsgListByConvIdReq {
  message Item {
    string convId = 1;
    repeated string seqList = 2;
  }
  repeated Item items = 2;
}
```

##### AckNotice

```protobuf
//AckNoticeDataReq 确认通知数据 如果不确认 服务端会一直推送 直到客户端消费成功
message AckNoticeDataReq {
  string noticeId = 2;
  string convId = 3;
}
```

##### GetMsgById

```protobuf
// 获取一条消息
message GetMsgByIdReq {
  optional string serverMsgId = 2;
  optional string clientMsgId = 3;
}
```

##### CustomRequest

```protobuf
// 自定义请求
message CustomRequestBody {
  string method = 1; // 类似于http的url
  bytes data = 2; // 类似于http的请求body
}
```

### 响应

```protobuf

enum PushEvent {
  // 消息推送
  PushMsgDataList = 0;
  // 通知推送
  PushNoticeDataList = 1;
  // 响应返回
  PushResponseBody = 2;
}

message PushBody {
  PushEvent event = 1;
  bytes data = 2;
}

// 服务端返回响应的消息体
message ResponseBody {
  enum Code {
    Success = 0;

    UnknownError = 1;
    InternalError = 2;
    RequestError = 3;
    AuthError = 4;
    ToastError = 5;
    AlertError = 7;
    RetryError = 8;
  }
  ActiveEvent event = 1;
  string reqId = 2;
  Code code = 3;
  bytes data = 4;
}
```

> 说明：服务端返回的消息体，其中 `event` 固定为 `PushResponseBody`，`data` 固定为 `ResponseBody` 的序列化数据。

#### ResponseBody说明

- event：请求事件类型
- reqId：请求id
- code：响应码
- data：响应数据

#### ResponseBody.data说明

##### SendMsgList

```protobuf
message SendMsgListResp {}
```

##### SyncConvSeq

```protobuf
// 同步会话消息seq
message BatchGetConvSeqResp {
  message ConvSeq {
    string convId = 1;
    string minSeq = 2;
    string maxSeq = 3;
    string updateTime = 4;
  }
  map<string, ConvSeq> convSeqMap = 2;
}
```

##### SyncMsgList

```protobuf
// 同步消息
message GetMsgListResp {
  repeated MsgData msgDataList = 2;
}
```

##### AckNotice

```protobuf
message AckNoticeDataResp {}
```

##### GetMsgById

```protobuf
// 获取一条消息
message GetMsgByIdResp {
  MsgData msgData = 2;
}
```

##### CustomRequest

```protobuf
// 根据method不同，data的结构也不同 具体应该查询 api 文档
```