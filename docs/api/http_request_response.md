# http api 请求以及响应

## 1. 请求

### 1.1 请求头中的参数

| 参数名 | 说明                                                 | 是否必须 | 默认值                    |
| :--- |:---------------------------------------------------|:-----|:-----------------------|
| Content-Type | 请求的数据类型                                            | 是    | application/x-protobuf |

### 1.2 请求方法和路径

    请求方法固定为POST
    请求路径格式为: /{version}/{service}/[isNeedAuth]/{method}

### 1.3 请求体

    请求体中的数据类型为`protobuf`，需要传递 commonReq 的请求体，其中data字段传具体的protobuf。

```protobuf
message CommonReq {
  string id = 1;
  string token = 2;

  string deviceModel = 11;
  string deviceId = 12;
  string osVersion = 13;
  string platform = 14;

  string appVersion = 21;
  string language = 22;

  bytes data = 31;

  string ip = 41; // 不需要传递
  string userAgent = 42; // 不需要传递
}
```

## 2. 响应

### 2.1 http状态码

| 状态码 | 说明         |
| :--- |:-----------|
| 200  | 请求成功       |
| 400  | 请求参数错误     |
| 401  | 未授权         |
| 500 | 服务器内部错误 |

### 2.2 响应体

    响应体中的数据类型为`protobuf`，以下是proto定义的消息体，其中`data`字段也是一个protobuf消息体，具体的消息体定义在各个接口的proto文件中。

```protobuf
message CommonResp {
  enum Code {
    Success = 0;

    InternalError = 2; // 内部错误
    RequestError = 3;  // 请求错误
    ToastError = 5;    // toast 错误 只有 message；此时 msg 为错误信息
    AlertError = 7;    // alert 错误 默认只有一个确定按钮；此时 msg 为 `{title: '标题', msg: '内容', actions: [{action: 0, title: '确定', jumpTo: ''}]}`
    RetryError = 8;    // alert 错误 有一个取消按钮 和一个重试按钮
  }
  Code code = 1;
  optional string msg = 2;
  bytes data = 3;
}
```
