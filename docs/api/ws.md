# websocket gateway

## 长连接请求和响应协议

### 请求

```protobuf
// 客户端发送的消息体
message RequestBody {
  string reqId = 1; // 客户端生成的请求id，返回时会原样返回
  string method = 2; // 请求方法，类似于http中的path
  bytes data = 3; // 请求数据
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
// 服务端会推送的消息体
message PushBody {
  PushEvent event = 1; // 此时 event 为 PushResponseBody 
  bytes data = 2; // 此时 data 为 ResponseBody
}
// 服务端返回响应的消息体
message ResponseBody {
  enum Code {
    Success = 0;

    UnknownError = 1;
    InternalError = 2; // 服务端内部错误
    RequestError = 3; // 请求错误 可能 method 不正确
    AuthError = 4; // 鉴权错误 可能没有登录或者登录过期
    ToastError = 5; // toast错误 需要toast提示
    AlertError = 7; // alert错误 需要alert提示
    RetryError = 8; // 服务端临时错误，需要重试
  }
  string reqId = 1; // 客户端生成的请求id，返回时会原样返回
  string method = 2; // 请求方法，类似于http中的path
  Code code = 3; // 响应码
  bytes data = 4; // 响应数据
}
```

## 通讯流程

### 1. 连接

> 客户端连接websocket，url为`ws[s]://<host>:<port>/ws`

### 2. 服务端返回

> 服务端推送一条字符串类型的消息，内容为`connected`

### 3. 设置连接参数

> 发送一条二进制类型消息，内容为RequestBody的序列化数据。其中`method`为`/v1/conn/setCxnParams`，`data`为`SetCxnParamsReq`的序列化数据

```protobuf
message CxnParams {
  string platform = 1;
  string deviceId = 2;
  string deviceModel = 3;
  string osVersion = 4;
  string appVersion = 5;
  string language = 6;
  string networkUsed = 7;
  bytes ex = 11;
}
message SetCxnParamsReq {
  CxnParams cxnParams = 1;
}

message SetCxnParamsResp {}
```

### 4. 设置用户参数

> 发送一条二进制类型消息，内容为RequestBody的序列化数据。其中`method`为`/v1/conn/setUserParams`，`data`为`SetUserParamsReq`的序列化数据

```protobuf
message SetUserParamsReq {
  string userId = 1;
  string token = 2;
  bytes ex = 11;
}

message SetUserParamsResp {}
```

### 5. 进行业务请求

发送一条二进制类型消息，内容为RequestBody的序列化数据。其中`method`为http api文档中的path，`data`为文档中的请求数据的序列化数据

> 需要注意的是，data中CommonReq不需要传递，此参数为服务端专用参数，客户端不需要传递 

#### 示例1. 发送消息

> 根据http api中msg文档，发送一条二进制类型消息，内容为RequestBody的序列化数据。其中`method`为`/v1/msg/sendMsgList`，`data`为`SendMsgListReq`的序列化数据

```protobuf
message SendMsgListReq {
  CommonReq commonReq = 1;
  repeated MsgData msgDataList = 2;
  // options
  //  延迟时间（秒） 不得大于 864000秒 也就是10天 只有开启了Pulsar的延迟消息功能才有效
  optional int32 deliverAfter = 11;
}

message SendMsgListResp {
  CommonResp commonResp = 1;
}
```